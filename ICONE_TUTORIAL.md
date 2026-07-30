# Tutorial: como trocar o ícone do app TIKA

Este tutorial assume que você **nunca personalizou um ícone de app
Flutter antes**. Vamos com calma, passo a passo.

O projeto já vem com um ícone temporário (gerado automaticamente,
em `assets/icon/`) só para o app não ficar sem ícone nenhum. Este guia
mostra como trocar por um ícone seu.

---

## 1. Prepare a imagem do ícone

- **Formato:** PNG.
- **Resolução:** quadrada, **1024×1024 pixels** (dá uma boa margem de
  qualidade; o mínimo aceitável seria 512×512, mas prefira 1024×1024).
- **Fundo:** pode ser uma imagem com fundo colorido normal (é o que vai
  virar o ícone "clássico"). Se você também quiser o efeito de **ícone
  adaptativo** (aquele que o Android recorta em círculo, quadrado
  arredondado, etc. dependendo do celular), você vai precisar de uma
  **segunda imagem**, com fundo **transparente**, contendo só o
  personagem/desenho, sem cortar as bordas — essa é a "camada de
  frente" (foreground). O projeto já está configurado para usar as
  duas.

Dica: se você só tem uma imagem (com fundo), pode usá-la para as duas
finalidades — não é perfeito (a versão adaptativa pode cortar um
pouco as bordas), mas funciona.

---

## 2. Onde colocar a imagem

Dentro da pasta do projeto, vá até:

```
assets/icon/
```

Substitua os dois arquivos que já existem lá (pode sobrescrever):

```
assets/icon/icon.png              ← ícone "clássico" (com fundo)
assets/icon/icon_foreground.png   ← camada de frente, fundo transparente
```

Se você só tem uma imagem, copie o mesmo arquivo com os dois nomes.

---

## 3. Configuração do `flutter_launcher_icons`

Essa parte **já está pronta** no `pubspec.yaml` do projeto — não
precisa mexer em nada, mas veja como funciona (é bom entender o que
está rodando):

```yaml
dev_dependencies:
  flutter_launcher_icons: ^0.13.1

flutter:
  uses-material-design: true
  assets:
    - assets/icon/

flutter_launcher_icons:
  android: true
  ios: false
  image_path: "assets/icon/icon.png"
  min_sdk_android: 21
  adaptive_icon_background: "#FFF4E0"
  adaptive_icon_foreground: "assets/icon/icon_foreground.png"
```

O que cada linha faz:

- `image_path`: qual imagem usar para o ícone clássico.
- `adaptive_icon_background`: cor sólida usada atrás da camada de
  frente no ícone adaptativo (você pode trocar por outra cor em
  hexadecimal, ou apontar para uma imagem também, se quiser).
- `adaptive_icon_foreground`: a imagem com fundo transparente.
- `min_sdk_android: 21`: garante compatibilidade com aparelhos Android
  mais antigos também.

Se no futuro você quiser gerar ícone para iOS também, basta trocar
`ios: false` para `ios: true` (o projeto atual está focado em Android,
conforme pedido).

---

## 4. Gerar os ícones

Com as imagens já substituídas em `assets/icon/`, rode no terminal,
dentro da pasta do projeto:

```bash
flutter pub get
dart run flutter_launcher_icons
```

Você deve ver uma mensagem final parecida com:

```
✓ Successfully generated launcher icons
```

---

## 5. Quais arquivos são modificados automaticamente

O comando acima **sobrescreve automaticamente** vários arquivos dentro
de `android/app/src/main/res/` — você não precisa (e não deve) editar
esses arquivos manualmente:

```
android/app/src/main/res/
├── mipmap-mdpi/ic_launcher.png
├── mipmap-hdpi/ic_launcher.png
├── mipmap-xhdpi/ic_launcher.png
├── mipmap-xxhdpi/ic_launcher.png
├── mipmap-xxxhdpi/ic_launcher.png
├── mipmap-anydpi-v26/ic_launcher.xml        ← aponta para o ícone adaptativo
├── mipmap-*/ic_launcher_foreground.png       ← camada de frente, em cada resolução
└── values/colors.xml (ou similar)            ← cor de fundo do ícone adaptativo
```

Cada pasta `mipmap-XXdpi` é uma resolução diferente, para telas com
densidades de pixel diferentes — o gerador cuida de redimensionar tudo
sozinho a partir da sua imagem de 1024×1024.

---

## 6. Como verificar se a alteração foi aplicada corretamente

1. Rode `flutter run` (ou reinstale o APK).
2. Veja o ícone na gaveta de apps do Android.
3. Se ainda estiver vendo o ícone antigo (aquele "A" azul padrão do
   Flutter, ou o ícone temporário anterior), veja a seção de problemas
   comuns abaixo — geralmente é cache.

---

## 7. Problemas comuns e como resolver

### O ícone antigo continua aparecendo

Isso quase sempre é cache — do Gradle, do launcher do Android, ou do
próprio dispositivo. Tente, nesta ordem:

```bash
# 1. Limpe o build do Flutter/Gradle
flutter clean

# 2. Baixe as dependências de novo
flutter pub get

# 3. Gere os ícones de novo
dart run flutter_launcher_icons

# 4. Rode novamente
flutter run
```

Se mesmo assim persistir:

- **Desinstale o app do aparelho/emulador** antes de rodar `flutter run`
  de novo (o Android às vezes mantém o ícone antigo em cache mesmo após
  reinstalar por cima).
- No emulador, às vezes é preciso **reiniciar o launcher** (reiniciar o
  emulador resolve na maioria dos casos).

### Erro ao rodar `dart run flutter_launcher_icons`

- Confirme que os dois arquivos existem exatamente nesses caminhos:
  `assets/icon/icon.png` e `assets/icon/icon_foreground.png`.
- Confirme que são PNGs válidos (abra num visualizador de imagens para
  checar se não corromperam ao copiar).
- Confirme que rodou `flutter pub get` antes (o pacote
  `flutter_launcher_icons` precisa estar instalado).

### O ícone adaptativo aparece cortado estranho

Isso normalmente significa que o desenho na sua
`icon_foreground.png` está ocupando pixels demais perto da borda — o
Android recorta cerca de 33% das bordas dependendo do formato de
máscara do aparelho (círculo, quadrado arredondado, "squircle", etc.).
Deixe uma margem de segurança: mantenha o desenho principal dentro dos
66% centrais da imagem de 1024×1024.

---

Pronto! Com isso o ícone do TIKA já é totalmente seu.
