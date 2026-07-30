# Widget de tela inicial do Android (opcional/avançado)

O app já vem pronto do lado Flutter para alimentar um widget: sempre que
passos, humor ou o próximo lembrete mudam, basta chamar
`HomeWidgetService.updateWidgetData(...)` (em
`lib/data/services/home_widget_service.dart`) e os dados ficam disponíveis
para o widget nativo ler.

A parte que falta é o **widget Android em si**, que é código nativo
(Kotlin + XML) e só pode ser criado depois que as pastas `android/` do
projeto existirem — ou seja, depois do passo `flutter create .` descrito
no `README.md`.

Isto é opcional: o app funciona 100% sem essa parte. Trate isso como uma
melhoria a mais, não como um bloqueador.

> **Nota:** os nomes de pacote (`es.antonborri.home_widget`, etc.) abaixo
> refletem a API do pacote `home_widget` na época em que este projeto foi
> montado. Se o import não resolver depois de rodar `flutter pub get`,
> abra `~/.pub-cache/hosted/pub.dev/home_widget-*/example/android` (ou o
> cache equivalente do seu sistema) para conferir os imports exatos da
> versão instalada — plugins mudam de tempos em tempos.

## Passo 1 — Layout do widget

Crie `android/app/src/main/res/layout/tika_widget_layout.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/widget_root"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="#FFF4E0">

    <TextView
        android:id="@+id/widget_mood"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:textSize="28sp" />

    <TextView
        android:id="@+id/widget_steps"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="6dp"
        android:textColor="#2B2320"
        android:textSize="16sp"
        android:textStyle="bold" />

    <TextView
        android:id="@+id/widget_reminder"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="4dp"
        android:textColor="#7A6F68"
        android:textSize="13sp" />

</LinearLayout>
```

## Passo 2 — Metadados do widget

Crie `android/app/src/main/res/xml/tika_widget_info.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="180dp"
    android:minHeight="110dp"
    android:updatePeriodMillis="1800000"
    android:initialLayout="@layout/tika_widget_layout"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen" />
```

`updatePeriodMillis` é só um "empurrão" periódico do Android (mínimo de
30 minutos); as atualizações "de verdade" acontecem quando o app chama
`HomeWidgetService.updateWidgetData(...)`, que força o widget a redesenhar
na hora.

## Passo 3 — Provider em Kotlin

Crie `android/app/src/main/kotlin/<caminho_do_seu_pacote>/TikaHomeWidgetProvider.kt`
(o caminho de pastas deve espelhar o `applicationId` do
`android/app/build.gradle` — por padrão, algo como
`android/app/src/main/kotlin/com/example/tika_app/`):

```kotlin
package com.example.tika_app // ajuste para o seu applicationId

import android.appwidget.AppWidgetManager
import android.content.Context
import android.widget.RemoteViews
import es.antonborri.home_widget.HomeWidgetLaunchIntent
import es.antonborri.home_widget.HomeWidgetProvider

class TikaHomeWidgetProvider : HomeWidgetProvider() {
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray,
        widgetData: android.content.SharedPreferences
    ) {
        appWidgetIds.forEach { widgetId ->
            val views = RemoteViews(context.packageName, R.layout.tika_widget_layout).apply {
                val steps = widgetData.getInt("tika_steps", 0)
                val goal = widgetData.getInt("tika_step_goal", 8000)
                val mood = widgetData.getString("tika_mood_emoji", "🙂")
                val reminder = widgetData.getString("tika_reminder_text", "Sem lembretes agora")

                setTextViewText(R.id.widget_steps, "$steps / $goal passos")
                setTextViewText(R.id.widget_mood, mood)
                setTextViewText(R.id.widget_reminder, reminder)

                val pendingIntent = HomeWidgetLaunchIntent.getActivity(context, MainActivity::class.java)
                setOnClickPendingIntent(R.id.widget_root, pendingIntent)
            }
            appWidgetManager.updateAppWidget(widgetId, views)
        }
    }
}
```

## Passo 4 — Registrar no AndroidManifest.xml

Dentro da tag `<application>` do
`android/app/src/main/AndroidManifest.xml`, adicione:

```xml
<receiver android:name=".TikaHomeWidgetProvider" android:exported="false">
    <intent-filter>
        <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
    </intent-filter>
    <meta-data
        android:name="android.appwidget.provider"
        android:resource="@xml/tika_widget_info" />
</receiver>
```

## Passo 5 — Testar

1. `flutter run` com o app em um dispositivo/emulador.
2. Segure o dedo na tela inicial do Android → Widgets → procure "TIKA".
3. Arraste o widget para a tela inicial.
4. Abra o app, mude algo (passos, humor) e volte para a tela inicial —
   o widget deve atualizar em poucos segundos.

Se o widget não atualizar, confirme que alguma tela do app está mesmo
chamando `HomeWidgetService.updateWidgetData(...)` — no scaffold atual
isso fica como um próximo passo de integração (ver seção "Como adicionar
novas funcionalidades" no README).
