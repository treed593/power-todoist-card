# PowerTodoist Card

[![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg)](https://github.com/hacs/integration)
![hacs_badge](https://img.shields.io/github/v/release/grinstantin/todoist-card)
![hacs_badge](https://img.shields.io/github/license/grinstantin/todoist-card)

PowerTodoist card for [Home Assistant](https://www.home-assistant.io) Lovelace UI. This card displays items from selected Todoist projects.

![Preview of todoist-card](https://user-images.githubusercontent.com/34913257/108243361-a8ea8500-7156-11eb-8313-a149a7cf38b8.png)

## Installing

### HACS

This card is available in [HACS](https://hacs.xyz) (Home Assistant Community Store).

Just search for `PowerTodoist Card` in HACS `Frontend` tab.

### Manual

1. Download `powertodoist-card.js` file from the [latest release](https://github.com/pgorod/power-todoist-card/releases/latest).
2. Put `powertodoist-card.js` file into your `config/www` folder.
3. Add a reference to `powertodoist-card.js` in Lovelace. There's two way to do that:
   1. **Using UI:** _Configuration_ → _Lovelace Dashboards_ → _Resources_ → Click Plus button → Set _Url_ as `/local/todoist-card.js` → Set _Resource type_ as `JavaScript Module`.
   2. **Using YAML:** Add the following code to `lovelace` section.
      ```yaml
      resources:
        - url: /local/powertodoist-card.js
          type: module
      ```
4. Add `custom:powertodoist-card` to Lovelace UI as any other card (using either editor or YAML configuration).

## Using the card

This card can be configured using Lovelace UI editor.

1. Add the following code to `configuration.yaml`:
    ```yaml
    sensor:
      - platform: rest
        name: To-do List
        method: GET
        resource: 'https://api.todoist.com/sync/v9/projects/get_data'
        params:
          project_id: TODOIST_PROJECT_ID
        headers:
          Authorization: !secret todoist_api_token
        value_template: '{{ value_json[''project''][''id''] }}'
        json_attributes:
          - project
          - items
        scan_interval: 30

    rest_command:
      todoist:
        method: post
        url: 'https://api.todoist.com/sync/v9/{{ url }}'
        payload: '{{ payload }}'
        headers:
          Authorization: !secret todoist_api_token
        content_type: 'application/x-www-form-urlencoded'
    ```
2. ... and to `secrets.yaml`:
    ```yaml
    todoist_api_token: 'Bearer TODOIST_API_TOKEN'
    ```
3. Replace `TODOIST_API_TOKEN` with your [token](https://todoist.com/prefs/integrations) and `TODOIST_PROJECT_ID` with ID of your selected Todoist project.
    > You can get `TODOIST_PROJECT_ID` from project URL, which usually looks like this:
    `https://todoist.com/app/project/TODOIST_PROJECT_ID`
4. Reload configs or restart Home Assistant.
5. In Lovelace UI, click 3 dots in top left corner.
6. Click _Edit Dashboard_.
7. Click _Add Card_ button in the bottom right corner to add a new card.
8. Find _Custom: Todoist Card_ in the list.
9. Choose `entity`.
10. Now you should see the preview of the card!

A basic example of using this card in YAML config could look like this:

```yaml
type: 'custom:powertodoist-card'
entity: sensor.to_do_list
show_header: true
show_completed: 5
use_quick_add: true
```
Note that the `to_do_list` entity name is what Home Assistant created based on the `name: To-do List` you specified earlier.

### Configuration Options

| Name                 |   Type    |   Default    | Description     |
| -------------------- | :-------: | :----------: | ------------------------------------------------------------------ |
| `type`               | `string`  | **required** | `custom:todoist-card`            |
| `entity`             | `string`  | **required** | An entity_id within the `sensor` domain.   |
| `show_completed`     | `integer` | `5`          | Number of completed tasks shown at the end of the list (0 to disable).   |
| `show_header`        | `boolean` | `true`       | Show friendly name of the selected `sensor` in the card header.      |
| `show_item_add`      | `boolean` | `true`       | Show text input element for adding new items to the list.        |
| `show_item_close`    | `boolean` | `true`       | Show `close/complete` and `uncomplete` buttons.       |
| `show_item_delete`   | `boolean` | `true`       | Show `delete` buttons.        |
| `show_card_labels`   | `boolean` | `true`       | Show card-level labels on top (see `filter_labels` below).        |
| `use_quick_add`      | `boolean` | `false`      | Use the [Quick Add](https://todoist.com/help/articles/task-quick-add) implementation, available in the official Todoist clients. |
| `sort_by_due_date`   | `boolean` | `false`      | Sort the tasks by their due date.   |
| `friendly_name`      | `string` | `Todoist`      | The card name shown on top uses a somewhat elaborate logic: <br>the default is `Todoist`, if no name is specified. <br>But if a Section filter is specified, then that section name will be used instead. <br>Finally, if you do use the `friendly_name` option, it will override anything else.  |
| `icons`   | `list` | ![image](https://github.com/pgorod/power-todoist-card/assets/15945027/793f8b01-4203-4e5a-81e1-785bb1d284a3)![image](https://github.com/pgorod/power-todoist-card/assets/15945027/2753b1ac-8e5a-42d7-b39d-6b30fb8fea2c)![image](https://github.com/pgorod/power-todoist-card/assets/15945027/5714d361-376b-4666-9ebf-a7611b13d0d4)![image](https://github.com/pgorod/power-todoist-card/assets/15945027/cc682901-ca9b-43e7-b1ba-e7eb276f66b4) | A list of 4 icon names from the [MDI Library](https://pictogrammers.com/library/mdi/), without the `mdi:`prefix. Icons will be used for the check mark of checkable items, the bullet of uncheckable items, the plus sign to add items, and the trash can to delete. <br>Defaults as shown are `["checkbox-marked-circle-outline", "circle-medium", "plus-outline", "trash-can-outline"]` |
| `filter_today_overdue` | `boolean` | `false`      | Only show tasks that are overdue or due today.    |
| `filter_due_days_out`       | `integer` | `-1`      | Show only items which have a due date within N days into the future. Using -1 disables this filter.   |
| `filter_section_id` | `integer` | `(none)`      | Only show tasks from one Todoist section, identified by its id.    |
| `filter_section` | `string` | `(none)`      | Only show tasks from one Todoist section, identified by its name.  |
| `filter_labels` | `list` | `(none)`      | Only show tasks with the specified Todoist labels. See below for details on this powerful option.    |

> Note that the completed tasks list is cleared when the page is refreshed.

#### Filtering by labels

Labels are colorful and beautiful. And useful! Define them in Todoist app, and don't forget to pick your favorite colors there - after some time, they will be picked up by PowerTodoist-card!

The `filter_labels` option allows for several possibilities.
- a label name will **include** items with that label
- if you prefix a label with `!` then it will **exclude** items with that label.
- use a special label called `*` to include items with any label (but not items without any labels at all).
- use a special label called `!*` to include items without any labels at all.
- you can combine lines with any of the above filters to join these conditions (for nerds, this will be a logical OR of the inclusion filters, but the exclusion filters always defeat anything else).

| Filters                 |   Meaning |
| --------------------------------------------------- | ------------------------------------------------- |
| `filter_labels:`<br>`  - "!Hidden"`<br>`  - "%user%"`<br>`  - "Important"`    | Shows items that don't have the `Hidden` label, and that either have <br>the `Important` label or a label with the current HASS user name. |

When you filter by a single label, that label won't appear graphically under each item; instead, it will appear on the top, next to the list name.

## Actions

- _Circle_ marks selected task as completed.
- _Plus_ "uncompletes" selected task, adding it back to the list.
- _Trash bin_ deletes selected task (gray one deletes it only from the list of completed items, not from Todoist archive).
- _Input_ adds new item to the list after pressing `Enter`.

## To-do / Things you can help with!

- many options are only available through YAML configuration, but could easily be added to the **UI configuration**. If only somebody would do this and test properly and create a PR... for the most part, you'd just have to copy existing code for the other options.
- ability to set **icon colors**
- I'd love to have actions you can trigger with **long taps**. But this is out of my Javascript league, I am not a JS guy, this is the first project I've done in JS, and I can't even say that I've learned it, I just stumbled along and did it.

## License

This project is licensed under the MIT license.

Copyright for portions of project PowerTodoist-card are held by [Konstantin Grinkevich](https://github.com/grinstantin) as part of project Todoist-card. All other copyright for project PowerTodoist-card are held by [pgorod](https://github.com/pgorod)

# Sponsor me, please!

If you enjoy and use this card, I'd appreciate it if you can sponsor my work. I'm actually trying to make a living from Github sponsorships, mostly from other projects, but Home Assistant users are numerous, every small donation will also help! Thanks, I really appreciate it!

[![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/pgorod)
