# Description 

This patch handles the installation of the extension once kiwi is launched on an android device. 

It disables the ui prompt that is shown when the user wants to install an extension. 

For the sake of brevity, the code that was changed will not be posted in this document. 

It can be looked up by grepping for the strings *"TELE"* and *"KIWI"*


## Files changed

### File: extension.h

in `extensions\common\extension.h`

### File: extension.cc

in `extensions\common\extension.cc`

### File: extension_install_prompt.cc

in `./chrome/browser/extensions/extension_install_prompt.cc`

### File: extension_service.cc

in `./chrome/browser/extensions/extension_service.cc`

### File: standard_management_policy_provider.cc

in `./chrome/browser/extensions/standard_management_policy_provider.cc`
