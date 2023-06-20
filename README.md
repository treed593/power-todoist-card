# PowerTodoist Card

[![hacs_badge](https://img.shields.io/badge/HACS-Default-orange.svg)](https://github.com/hacs/integration)
![hacs_badge](https://img.shields.io/github/v/release/pgorod/power-todoist-card)
![hacs_badge](https://img.shields.io/github/license/pgorod/power-todoist-card)

PowerTodoist card for [Home Assistant](https://www.home-assistant.io) Lovelace UI. This card displays items from selected Todoist projects.

https://github.com/pgorod/power-todoist-card/assets/15945027/d049b00a-d406-4b7c-8803-92695d88b1b2

## Installing

### HACS

(not working yet...)

~~This card is available in [HACS](https://hacs.xyz) (Home Assistant Community Store).~~

~~Just search for `PowerTodoist Card` in HACS `Frontend` tab.~~

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
      - name: To-do List
        platform: rest
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
          - sections
          - project_notes
        scan_interval: 30
        
      - name: label_colors
        platform: command_line
        command: !secret todoist_cmd_with_api_token
        value_template: > 
            {{ value_json.label_colors | length }}
        json_attributes:
            - label_colors 
        scan_interval: 200

    rest_command:
      todoist:
        method: post
        url: 'https://api.todoist.com/sync/v9/{{ url }}'
        payload: '{{ payload }}'
        headers:
          Authorization: !secret todoist_api_token
        content_type: 'application/x-www-form-urlencoded'
    ```
    👉 The REST command and the `label_colors` sensor are constant and need to be defined only once for each Todoist account used (I recommend using only one and handling any content separation with cleverly filtered projects, sections, and labels).
    
    👉 The Sensor definition, on the other hand, can be cloned to allow for different projects, just make sure you set a unique entity name, and set the appropriate `TODOIST_PROJECT_ID` for each one (see below).
2. In that `configuration.yaml`, replace `TODOIST_PROJECT_ID` with ID of your selected Todoist project.
    > You can get `TODOIST_PROJECT_ID` from project URL, which usually looks like this:
    `https://todoist.com/app/project/TODOIST_PROJECT_ID`
3. Add this to `secrets.yaml`:
    ```yaml
    todoist_api_token: 'Bearer TODOIST_API_TOKEN'
    todoist_cmd_with_api_token: 'echo "{\"label_colors\":" $(curl -s https://api.todoist.com/rest/v2/labels -H "Accept: application/json" -H "Authorization: Bearer TODOIST_API_TOKEN") "}" '
    ```
4. In that `secrets.yaml`, replace two instances of `TODOIST_API_TOKEN` with your private [API Token](https://todoist.com/prefs/integrations) (click `Developer` tab on that page).
5. Reload configs or restart Home Assistant.
6. In Lovelace UI, click 3 dots in top-right corner.
7. Click _Edit Dashboard_.
8. Click _Add Card_ button in the bottom right corner to add a new card.
9. Find _Custom: PowerTodoist Card_ in the list.
10. Choose `entity`.
11. Now you should see the preview of the card!

A basic example of using this card in YAML config could look like this:

```yaml
type: 'custom:powertodoist-card'
entity: sensor.to_do_list
show_header: true
show_completed: 5
use_quick_add: true
```
Note that the `to_do_list` entity name is what Home Assistant created based on the `name: To-do List` you specified earlier in the sensor definition. 
Spaces and hyphens turned into `_`, and everything became lowercase. In case of doubt search your entities in HASS for the correct name.

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

| Label Filter examples                 |   Meaning |
| --------------------------------------------------- | ------------------------------------------------- |
| `filter_labels:`<br>`  - "Blue"`<br>`  - "Green"`    | Shows items with either a `Green` or a `Blue` label, or both. |
| `filter_labels:`<br>`  - "!Done"`   | Shows items that don't have a `Done` label. |
| `filter_labels:`<br>`  - "!Hidden"`<br>`  - "%user%"`<br>`  - "Important"`    | Shows items that don't have the `Hidden` label, and that either have <br>the `Important` label or a label with the current HASS user name. |
| `filter_labels:`<br>`  - "For%user%"`    | Shows items that have a label composed of the letters `For` and the current user name. For example, if user is called `Joe` then a label `ForJoe` would match. |

When you filter by a single label, that label won't appear graphically under each item; instead, it will appear on the top, next to the list name.

## Handling Events with Actions

Basic behaviour is this:
- ![image](https://github.com/pgorod/power-todoist-card/assets/15945027/793f8b01-4203-4e5a-81e1-785bb1d284a3) marks selected task as completed.
- ![image](https://github.com/pgorod/power-todoist-card/assets/15945027/5714d361-376b-4666-9ebf-a7611b13d0d4)  "uncompletes" selected task, adding it back to the list.
- ![image](https://github.com/pgorod/power-todoist-card/assets/15945027/cc682901-ca9b-43e7-b1ba-e7eb276f66b4) deletes selected task (gray one deletes it only from the list of completed items, not from Todoist archive).
- _Input_ adds new item to the list after pressing `Enter`.

But with this card, almost everything is programmable :-), so in the configurations you can optionally create lists of custom actions to run when the user does something. These action lists use the following headers according to the user event:

| **User Event**               |   Triggered by | Default actions (if no custom actions are specified) |
| --------------------------------------------------- | ------------------------------------------------- | --------- |
| **actions_close** | A tap on the close button (checking a task as done) | Close (complete) the task as done in Todoist. |
| actions_dbl_close | A  double-tap on the close button (checking a task as done) | Nothing |
| **actions_content** | A tap on the content (text) of the task | Open dialog box to update the text of the task. |
| actions_dbl_content | A double-tap on the content (text) of the task | Nothing |
| **actions_description** | A tap on the desciption text of the task, if it is defined in Todoist. | Open dialog box to update the description of the task. |
| actions_dbl_description | A double-tap on the desciption text of the task, if it is defined in Todoist. | Nothing |
| **actions_label** | A tap on any of the task's labels (currently it is not possible to differentiate taps on different labels). | Nothing |
| actions_dbl_label | A double-tap on any of the task's labels (currently it is not possible to differentiate taps on different labels). | Nothing |
| **actions_delete** | A tap on the delete button to the right of the task. | Delete the task in Todoist. |
| actions_dbl_delete | A double-tap on the delete button to the right of the task. | Nothing |
| **actions_uncomplete** | A tap on the task uncomplete button on a recently completed task (shown at the bottom). | Undo the task completion of a recently completed task. |
| actions_dbl_uncomplete | A double-tap on the task uncomplete button on a recently completed task (shown at the bottom). | Nothing |

Under those headers, you can use a list of actions to specify custom behaviours. Your options for the actions are:

| **Custom Action**                 |   What it does |
| -------------------------------------- | ----------------------------------------- |
| **`close`**        | Closes (completes) the item in Todoist. |
| **`delete`**       | Deletes the item in Todoist. |
| **`move`**         | Moves the item in Todoist to the next section. If it's currently in the last section, item will be moved to the special "(No section)" section. |
| **`confirm`**      | Opens a dialog box asking the user to confirm the action. If the user approves, action continues, otherwise it is cancelled. <br>Specify an optional string for a custom confirmation text. |
| **`toast`**        | Flashes a message for a few seconds giving some information to the user. |
| **`update`**       | Sets new values to the item's fields in Todoist. <br>This is a list which allows all Todoist field types: `content`, `description`, `priority`, `collapsed`, `assigned_by_uid`, `responsible_uid`, `day_order`<br>You can use variables here. One of the variables can be `%input%` which will cause the user to be prompted for the value. use `prompt_texts` to control the dialog box displayed. |
| **`prompt_texts`** | Not really an action, but you can use this to set a couple of strings to be used in other actions. <br>Use two strings separated by the `\|` character. The first will be the dialog box title, the second will be the default value presented in the input box. The variable `%content%` will be useful here to show the current value. <br>So, for example, `"Insert new name:\|%content%"` will set the prompt for an update action that references `%input%` , asking the user to give a new name, showing the current name in the input box, so it can be easily edited. |
| **`automation`**   | Runs the specified automation in Home Assistant. This is very powerful! It is the action that contains a thousand possibilities! 🚀 <br>And this is what makes the PowerTodoist card really a Home Assistant thing - not just an interface to your Todoist. |
| **`label`**        | Here `label` is a verb! You use this action to label the current item as you prefer. <br>Use a list of label names. If you prefix a label name with a `!` that label will be removed, instead of added. Use `!*` to clear all labels. |
| **`add`**          | A list of texts that will be sent as **Quick Add** to Todoist. More details about the syntax [below](#adding_tasks_from_automation). Remember to specify Project, section and labels, as necessary to match your filters and ensure the new item is visible where you want it. |
| **`allow`**        | A list of user names that will be allowed to execute all the actions in the handling of the current event. To allow everyone, simply don't use this option. |

The items in this **Actions** table must always appear below a user **Event** from the previous table, like this:
```
- actions_delete
  - confirm
  - delete
```
Actions are executed in the order given. If you include a list of actions for an event, the default actions won't be executed unless you specify them explicitly. 

## Variables

This feature is undergoing a major re-factoring and might not work perfectly well in all places. But the general idea is that you can use these in any configuration option and they will get substituted, if the value is available (makes sense in that context). In case of doubt, just try it and see how it goes.

| **Variable name**                 |   Text that will be substituted | 
| -------------------------------------- | ----------------------------------------- |
| **`%was%`**   | Previous value |
| **`%input%`**   | Value returned from a prompt shown to the user. The prompt can be customized with `prompt_texts` action. |
| **`%line%`**   | A new line character. Allows breaking lines in places where it would be difficult to add a new line in YAML. |
| **`%user%`**   | The current Home Assistant user name. This is also super powerful to build multi-user systems. You can create separate lists per user, and you can use labels with user names to move things from one to the other. Note that this is not the Todoist user names; it is meant for use with a single Todoist user, the trick is to use labels or sections to separate tasks by HASS users. |
| **`%date%`**   | The current date, formatted as specified by the `date_format` option. <br>Default format is `"mmm dd H:mm"`<br>Complete formatting options are documented [here](https://blog.stevenlevithan.com/archives/javascript-date-format). |
| **`%str_labels%`**   | The current items labels, all concatenated as a comma-separated string. |

Let me know if you have other ideas for variables that could prove useful.

## Learn by example

Have a look at this demo file: 
https://github.com/pgorod/power-todoist-card/blob/main/todoist_demo.yaml

## Adding Tasks from Automation

- Create an automation with whatever triggers and conditions you want. 
- For the **actions**, use **Call Service** 
- Then you should have an option available called **`RESTful command: todoist`** (internally, `rest_command.todoist` which you defined in `configuration.yaml`). 
- Enter the following in the **Service data** box:
```
service: rest_command.todoist
data:
  url: quick/add
  payload: "text=Auto created at ({{ now().strftime('%H:%M') }}) #myproject @mylabel"
```
You can use the full power of Todoist's Quick add feature that will parse the text of the task to be created. It can include...
- due date (in free form text)
- #project
- @label
- +assignee
- /section
- // description (at the end)
- p2 priority

To get this right, I suggest you try your text first directly in the Todoist app, before moving it into the HASS automation.

You can also use the power of Home Assistant's templating language enclosed in `{{ }}` brackets, as my example shows.

## Kanbans and Multiple task lists

You can use as many of these cards as you want in your dashboards, of course. But I find that this looks particularly cool, and works in a particularly functional way, if you combine this card with the HACS [Swipe card](https://github.com/bramkragten/swipe-card). Just have a card for each Todoist section, and put them all together in a Swipe card, to get the feeling of a Kanban. Use the `move` action to move items to the next card.

Or use one card per user (family member) for home tasks that you want everyone to be able to see. You can check everybody's progress by swiping sideways. I even like to use a final card which is a Markdown card with home rules or instructions for the use of the Task list. 

## To-do / Things you can help with!

- many options are only available through YAML configuration, but could easily be added to the **UI configuration**. If only somebody would do this and test properly and create a PR... for the most part, you'd just have to copy existing code for the other options.
- ability to set **icon colors**
- I'd love to have actions you can trigger with **long taps**. But this is out of my Javascript league, I am not a JS guy, this is the first project I've done in JS, and I can't even say that I've learned it, I just stumbled along and did it :-)
- Automations can add Todoist tasks (with all the power of the Quick Add feature). But they can't change existing tasks, which would definitely be useful. This would require some way of focusing in on a specific task, since you don't have such context in the Automations. I haven't thought of a nice clean way of achieving this.

## License

This project is licensed under the MIT license.

Copyright for portions of project PowerTodoist-card are held by [Konstantin Grinkevich](https://github.com/grinstantin) as part of project Todoist-card. All other copyright for project PowerTodoist-card is held by [pgorod](https://github.com/pgorod)

# Sponsor me, please!

That was a lot of work 😅! If you enjoy and use this card, I'd appreciate it if you can sponsor my work. I'm actually trying to make a living from Github sponsorships, mostly from other projects, but Home Assistant users are numerous, every small donation will also help! Thanks, I really appreciate it!

[![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/pgorod)

Note that you pick a one-time amount and select any value you want.
