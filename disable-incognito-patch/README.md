# Description 

This patch disables the incognito option in chromium since it could be used to disable the extension

For the sake of brevity, the code that was changed will not be posted in this document. 

It can be looked up by grepping for the strings *"TELE"* and *"KIWI"*


## Files changed

### File: accelerator_table.cc

in `chrome\browser\ui\views\accelerator_table.cc`

### File: app_menu_model.cc

in `browser\ui\toolbar\app_menu_model.cc`

### File: app_menu_model.cc

in `chrome\browser\ui\browser_command_controller.cc`

