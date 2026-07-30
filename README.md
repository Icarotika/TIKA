# TIKA — seu companheiro virtual

Um "digipet moderno" que vive no seu celular: acompanha sua rotina, ouve
você, comemora suas conquistas e nunca precisa ser alimentado, limpo ou
cuidado. Feito em Flutter, focado em Android, 100% offline.

---

## Índice

1. [Visão geral](#visão-geral)
2. [Estrutura de pastas](#estrutura-de-pastas)
3. [Bibliotecas utilizadas e por quê](#bibliotecas-utilizadas-e-por-quê)
4. [Como executar o projeto](#como-executar-o-projeto)
5. [Como gerar APK e AAB](#como-gerar-apk-e-aab)
6. [Como adicionar novas funcionalidades](#como-adicionar-novas-funcionalidades)
7. [Limitações conhecidas / próximos passos](#limitações-conhecidas--próximos-passos)
8. [Tutorial: como trocar o ícone do app](#tutorial-como-trocar-o-ícone-do-app) → veja `ICONE_TUTORIAL.md`

---

## Visão geral

O app é dividido em 5 abas (Início, Hábitos, Missões, Estatísticas,
Ajustes), com Chat, Diário, Lembretes, Conquistas e Coleção acessíveis a
partir da Home. O personagem TIKA fica sempre visível na Home, animado
com `CustomPainter` puro (sem imagens externas) — pisca, respira, olha
para os lados, acena, comemora e dorme de madrugada.

**Nenhum dado sai do aparelho.** Tudo é persistido localmente com Hive.

---

## Estrutura de pastas

```
lib/
├── main.dart                 # bootstrap: Hive, notificações, intl
├── app.dart                  # MaterialApp + tema + tracking de tempo de uso
│
├── core/                     # Nada aqui depende do resto do app
│   ├── theme/                # cores, tipografia, ThemeData claro/escuro
│   ├── constants/             # nomes de boxes, canais de notificação,
│   │                           # e o "banco de falas" do TIKA (tika_phrases.dart)
│   └── utils/                 # datas, geração de id
│
├── domain/                   # Regras de negócio puras — zero Flutter, zero Hive
│   ├── entities/              # Habit, Mission, Reminder, MoodEntry, etc.
│   ├── repositories/          # CONTRATOS (interfaces) que a camada data implementa
│   └── usecases/              # Uma classe por ação de negócio relevante
│                               # (ex.: CompleteHabitUseCase, SendMessageUseCase)
│
├── data/                      # Implementação concreta dos contratos do domínio
│   ├── models/                 # Classes + TypeAdapter do Hive (uma por entidade)
│   ├── datasources/            # HiveBoxes: abre/registra todas as boxes
│   ├── repositories/           # *RepositoryImpl — convertem Model <-> Entity
│   └── services/                # Notificações, sensor de passos, engine de chat,
│                                 # ponte para o widget Android
│
└── presentation/              # Tudo que é UI
    ├── providers/               # Riverpod: liga UI aos use cases/repositórios
    ├── widgets/                  # Componentes reutilizáveis (TikaCharacter,
    │                              # SectionCard, ProgressRing, MoodSelector...)
    └── screens/                  # Uma pasta por tela
```

A regra de dependência é sempre **para dentro**:
`presentation → domain ← data`. `domain` nunca importa nada de `data` ou
`presentation`. Isso é o que torna, por exemplo, trivial trocar o motor
de conversa por uma IA real no futuro (veja a seção de novas
funcionalidades) sem tocar em nenhuma tela.

---

## Bibliotecas utilizadas e por quê

| Pacote | Por quê |
|---|---|
| `flutter_riverpod` | Gerenciamento de estado moderno, testável, e que combina bem com Clean Architecture (cada usecase/repositório vira um `Provider`, cada tela só `watch`a o que precisa). |
| `hive` + `hive_flutter` | Banco local rápido, 100% Dart (sem dependência nativa complexa), perfeito para "funcionar totalmente offline". Os adapters foram escritos **à mão** (sem `build_runner`/`hive_generator`) para o projeto não depender de nenhum passo extra de geração de código. |
| `flutter_local_notifications` + `timezone` + `flutter_timezone` | Notificações locais agendadas com hora exata, respeitando o fuso horário real do aparelho. |
| `pedometer` + `permission_handler` | Leitura do sensor de passos do Android e pedido de permissão em runtime. |
| `fl_chart` | Gráfico semanal de passos na tela de Estatísticas. |
| `intl` | Formatação de datas em português (`dd de MMMM`, dia da semana, etc.). |
| `uuid` | Geração de ids únicos para hábitos, mensagens, lembretes etc. |
| `home_widget` | Ponte Flutter ↔ Widget Android (a parte nativa do widget é opcional — ver `widget_setup_guide.md`). |
| `flutter_launcher_icons` (dev) | Gera os ícones do app a partir de uma imagem única — ver `ICONE_TUTORIAL.md`. |

Nenhuma dessas bibliotecas depende de servidor/API externa: o app
continua funcional mesmo sem internet.

---

## Como executar o projeto

Pré-requisitos: [Flutter SDK](https://flutter.dev) instalado (canal
stable) e um dispositivo/emulador Android.

```bash
# 1. Descompacte o projeto e entre na pasta
cd tika_app

# 2. Gere as pastas de plataforma (android/, ios/, etc.) — o projeto
#    já vem só com o código Flutter (lib/) e o pubspec.yaml; o comando
#    abaixo detecta o projeto existente e cria apenas o que falta, sem
#    sobrescrever lib/ nem pubspec.yaml.
flutter create .

# 3. Aplique as permissões do Android — abra
#    android/app/src/main/AndroidManifest.xml e cole o conteúdo de
#    android_manifest_snippet.xml (instruções dentro do próprio arquivo)

# 4. Instale as dependências
flutter pub get

# 5. Rode o app em um dispositivo/emulador conectado
flutter run
```

Na primeira execução, o app vai pedir permissão de **notificações** e de
**reconhecimento de atividade** (para os passos) — aceite ambas para a
experiência completa. Sem elas, o resto do app continua funcionando
normalmente.

---

## Como gerar APK e AAB

```bash
# APK de release (para instalar direto em um aparelho / testar fora da loja)
flutter build apk --release

# Encontra-se em:
# build/app/outputs/flutter-apk/app-release.apk

# App Bundle (formato exigido pela Google Play Store)
flutter build appbundle --release

# Encontra-se em:
# build/app/outputs/bundle/release/app-release.aab
```

Para publicar na Play Store de verdade, antes de gerar o AAB você vai
querer configurar uma **chave de assinatura própria** (ao invés da chave
de debug padrão) — veja a
[documentação oficial do Flutter sobre assinatura de app](https://docs.flutter.dev/deployment/android#signing-the-app)
quando chegar nessa etapa.

---

## Como adicionar novas funcionalidades

Graças à Clean Architecture, adicionar algo novo segue sempre o mesmo
caminho, de dentro para fora:

1. **Domain**: crie a entidade (`domain/entities`) se for um novo
   conceito, e o(s) usecase(s) (`domain/usecases`) com a regra de
   negócio. Se precisar persistir algo, defina o contrato em
   `domain/repositories`.
2. **Data**: crie o `Model` + `TypeAdapter` do Hive
   (`data/models`) copiando o padrão dos existentes (dê um `typeId`
   novo e ainda não usado — confira os já usados em
   `data/datasources/hive_boxes.dart`), registre a nova box em
   `HiveBoxes.init()`, e implemente o repositório
   (`data/repositories`).
3. **Presentation**: exponha o repositório/usecase em
   `presentation/providers/providers.dart`, crie um `Notifier`/provider
   dedicado se a tela precisar de estado próprio, e construa a tela
   reaproveitando os widgets em `presentation/widgets`
   (`SectionCard`, `ProgressRing`, etc.) para manter a identidade visual.

### Exemplo concreto: plugar uma IA real no modo Conversa

O motor de respostas do TIKA já está isolado atrás de uma interface —
`ChatResponder`, em `domain/usecases/send_message_usecase.dart`. A
implementação atual (`TikaResponseEngine`, em
`data/services/tika_response_engine.dart`) é baseada em regras. Para
trocar por uma IA:

```dart
class AiChatResponder implements ChatResponder {
  @override
  String respond(List<ChatMessage> sessionHistory, String userMessage) {
    // chame sua API aqui (ex.: Anthropic, usando o histórico da sessão
    // como contexto) e retorne o texto da resposta.
  }
}
```

E troque uma única linha em `presentation/providers/providers.dart`:

```dart
final chatResponderProvider =
    Provider<ChatResponder>((ref) => AiChatResponder()); // era TikaResponseEngine()
```

Nada mais no app precisa mudar — telas, providers e persistência
continuam iguais.

> **Importante:** o `TikaResponseEngine` tem um bloco de prioridade
> máxima que reconhece sinais de risco (ideação suicida/autolesão) e
> responde com acolhimento + o contato do CVV (188, ligação gratuita
> 24h). Se você trocar por uma IA, **mantenha uma rede de segurança
> equivalente** — não remova essa proteção.

---

## Limitações conhecidas / próximos passos

- **Widget de tela inicial**: o lado Flutter (`HomeWidgetService`) está
  pronto, mas o widget Android nativo (Kotlin) precisa ser adicionado
  manualmente — é avançado e depende das pastas geradas por
  `flutter create .`. Passo a passo completo em `widget_setup_guide.md`.
- **Sensor de passos**: só funciona em aparelho físico com sensor de
  passos (emuladores geralmente não têm). Sem o sensor, o resto do app
  funciona normalmente, só a contagem automática fica em 0.
- **Alarmes exatos (Android 12+)**: os lembretes usam agendamento
  "exato", que no Android 12+ pede uma permissão especial
  (`SCHEDULE_EXACT_ALARM`, já incluída em `android_manifest_snippet.xml`).
  Se preferir não lidar com essa permissão, há uma alternativa de uma
  linha só documentada dentro do próprio snippet.
- **Modo Conversa**: por design, é baseado em regras/padrões (não uma
  IA), exatamente como pedido — mas já isolado atrás de uma interface
  para plugar IA real depois (ver seção anterior).

---

## Tutorial: como trocar o ícone do app

Veja o arquivo **`ICONE_TUTORIAL.md`**, com o passo a passo completo.
