# Quests Engine — misiones, sus pasos y el diario

Quests Engine lleva la cuenta de las tareas del jugador. Una misión tiene un nombre y una lista de pasos numerados; los escenarios la inician, la llevan al paso siguiente, le ponen etiquetas y la completan. El panel **Diario** muestra las misiones que el jugador ha tomado, y el juego guarda su estado.

Lo usan Cthangover (el despertar del prólogo y la casa vieja) y RPG Plus (las misiones del bosque).

## El diario

Se abre con el icono del diario en la barra de herramientas. Mientras el jugador no haya tomado ninguna misión, está vacío:

![El diario de RPG Plus al empezar: pestañas Activas y Completadas, «No hay misiones» en ambas páginas.](assets/journal-before.jpg)

En RPG Plus, el anciano del pueblo da la primera misión. Pregúntale dónde buscar:

![La casa del anciano: el héroe frente al anciano, dos respuestas a la derecha — «¿Puede un lobo apagar faroles?» y «¿Dónde lo encuentro?».](assets/elder-choice.jpg)

Tras la respuesta, la misión aparece en el diario:

![El diario tras la charla: «Una luz en el libro de cuentas» en la lista de activas; a la derecha «Entrada II de V», el primer paso tachado y el paso actual «Vence al lobo de la cueva del bosque del este. Encuentra el corazón de ámbar».](assets/journal-after.jpg)

- **Activas** reúne las misiones en curso y **Completadas** las terminadas. Las misiones aún no empezadas no aparecen nunca.
- La página derecha muestra la misión seleccionada: los pasos hechos van tachados, el paso actual queda marcado y los futuros siguen ocultos, así el diario no adelanta la historia.
- Las etiquetas que la misión ha ganado se muestran como pequeñas placas doradas bajo la entrada.

## Una definición de misión

Las misiones son archivos JSON de tipo `quest` en la carpeta `quests/` de un mod. Esta es la misión de las imágenes (de RPG Plus):

```json
{
  "Kind": "quest",
  "Items": [
    {
      "Id": "woodland_ward",
      "Name": "rpg_plus/quest/ward/name",
      "StatusToDescription": {
        "0": "rpg_plus/quest/ward/0",
        "1": "rpg_plus/quest/ward/1",
        "2": "rpg_plus/quest/ward/2",
        "3": "rpg_plus/quest/ward/3",
        "4": "rpg_plus/quest/ward/4"
      }
    }
  ]
}
```

`Name` y cada paso son claves de traducción. Los números son el **paso** de la misión: en el paso 1 se muestra el texto de `1` como actual y el de `0` tachado — «Entrada II de V» en la imagen.

## Mover una misión desde un escenario

La respuesta del anciano en RPG Plus inicia la misión y la pone en el paso 1:

```scenario
action quest.set_status quest_id=woodland_ward status=Progress
action quest.set_data_status quest_id=woodland_ward level=1
```

- `quest.set_status` — el estado de la misión: `NotStarted`, `Progress` (activa) o `End` (completada).
- `quest.set_data_status` — el paso actual; `level` es su número.
- `quest.add_tag` / `quest.remove_tag` con `tag=…` — recordar algo que ocurrió, por ejemplo qué objeto cogió primero el jugador.
- `quest.send_notification` — muestra al jugador un aviso sobre la misión; cambiar una misión no lo hace por sí solo.

## Comprobar una misión

Las comprobaciones de misiones funcionan dondequiera que el juego admita una condición: `when`, condiciones de respuesta, condiciones de escena y eventos de mapa.

```scenario
when "woodland_ward.DataStatus >= 2" -> :lantern
```

- `<quest>.IsActive` — la misión está en curso;
- `<quest>.Status` — `0` no empezada, `1` en curso, `2` completada;
- `<quest>.DataStatus` — el paso actual;
- `<quest>.hasTag("tag")`, `<quest>.notHasTag("tag")` — Cthangover las usa para elegir qué escenario de una escena se reproduce, por ejemplo `AwakeningQuest.hasTag("book_taken")`.

## Si algo falla

1. **La misión no está en el diario.** Su estado debe ser `Progress` (o `End` para la pestaña de completadas); fijar solo el paso no basta.
2. **El diario muestra una clave en vez de texto.** Añade la clave del paso a los archivos `locale` del mod.
3. **Una condición siempre es falsa.** El id de la misión está escrito distinto del `Id` del archivo de la misión; el registro avisa de una misión desconocida.
