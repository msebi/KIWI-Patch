# Description 

This patch copies the extension into the local storage of the kiwi app once the app is started. The extension itself is stored 

as a binary in file `kiwi_extension_bin.h`. The reason to this approach was motivated by the fact that downloading the app from a 

server on startup might flag the app as being unsafe on g-play. chromium and as such, kiwi, do not enable auto installing extensions

from the [chrome webstore](https://chrome.google.com/webstore)

The extension is stored as a byte array and was obtained by running [Bin2C](https://www.segger.com/free-utilities/bin2c/) on 

the the .crx archive of the extension. 



## Files changed 

### File: extension_function_histogram_value.h

in `extensions\browser\extension_function_histogram_value.h`

with changes:

```
// TELE-CODE
DEVELOPERPRIVATE_LOADKIWIUNPACKED = 1546,
// / TELE-CODE
```

### File: startup_browser_creator_impl.cc

in `chrome\browser\ui\startup\startup_browser_creator_impl.cc`

with changes:

```
// TELE-CODE
#include "extensions/browser/extension_function_dispatcher.h"
#include "extensions/browser/extension_function_registry.h"
#include "extensions/browser/extensions_browser_client.h"

void CallExtensionInstallFunctionFromRegistry() {
  constexpr char kCreationFailed[] = "Access to extension API denied.";
  const std::string function_name = "DeveloperPrivateLoadUnpackedKiwiFunction";

  scoped_refptr<ExtensionFunction> function =
      ExtensionFunctionRegistry::GetInstance().NewFunction(function_name);
  if (!function) {
    LOG(ERROR) << "Unknown Extension API - " << function_name;
    return;
  }


  if (!function->HasPermission()) {
    LOG(ERROR) << "Permission denied for " << function_name;
    function->RespondWithError(kCreationFailed);
    return;
  }

  function->Run();
}
// / TELE-CODE
```

### File: generated_api_registration.cc

in `out\Default\gen\chrome\browser\extensions\api\generated_api_registration.cc`

with changes: 

```
Check if this file is (re)generated on build. 

// TELE-CODE
// Similar to DeveloperPrivateLoadUnpackedFunction but with predefined path
{
  &NewExtensionFunction<DeveloperPrivateLoadUnpackedKiwiFunction>,
  DeveloperPrivateLoadUnpackedKiwiFunction::static_function_name(),
  DeveloperPrivateLoadUnpackedKiwiFunction::static_histogram_value(),
},    
// / TELE-CODE
```

### File: developer_private_api.h

in `chrome\browser\extensions\api\developer_private\developer_private_api.h`

with changes:

```
// TELE-CODE
// The profile-keyed service that manages the extension autoinstaller. 
class KiwiExtensionInstallService : public BrowserContextKeyedAPI {
 public:
  using UnpackedRetryId = std::string;

  static BrowserContextKeyedAPIFactory<KiwiExtensionInstallService>*
  GetFactoryInstance();

  // Convenience method to get the KiwiExtensionInstallService for a profile.
  static KiwiExtensionInstallService* Get(content::BrowserContext* context);

  explicit KiwiExtensionInstallService(content::BrowserContext* context);
  ~KiwiExtensionInstallService() override;

  // KeyedService implementation
  void Shutdown() override;

  friend class BrowserContextKeyedAPIFactory<KiwiExtensionInstallService>;

  // BrowserContextKeyedAPI implementation.
  static const char* service_name() { return "KiwiExtensionInstallService"; }
  static const bool kServiceRedirectedInIncognito = true;
  static const bool kServiceIsNULLWhileTesting = true;

  void RegisterNotifications();

  Profile* profile_;

  // Created lazily upon OnListenerAdded.
  std::unique_ptr<DeveloperPrivateEventRouter> developer_private_event_router_;

  base::WeakPtrFactory<KiwiExtensionInstallService> weak_factory_{this};

  DISALLOW_COPY_AND_ASSIGN(KiwiExtensionInstallService);
};

template <>
void BrowserContextKeyedAPIFactory<
    KiwiExtensionInstallService>::DeclareFactoryDependencies();
// / TELE-CODE
```

and 

```
// TELE-CODE
// This function handles the installation of the kiwi extension.
// it uses a predefined path for loading the extension. Next, store
// the extension as a binary blob (of the .crx), copy it to the current 
// app directory on android and call the installer which is defined 
// in this function 

class DeveloperPrivateLoadUnpackedKiwiFunction
    : public DeveloperPrivateChooseEntryFunction {
 public:
  DECLARE_EXTENSION_FUNCTION("developerPrivate.loadKiwiUnpacked",
                             DEVELOPERPRIVATE_LOADKIWIUNPACKED)
  DeveloperPrivateLoadUnpackedKiwiFunction();

 protected:
  ~DeveloperPrivateLoadUnpackedKiwiFunction() override;
  ResponseAction Run() override;

  // EntryPickerClient:
  void FileSelected(const base::FilePath& path) override;
  void FileSelectionCanceled() override;

  // Callback for the UnpackedLoader.
  void OnLoadComplete(const Extension* extension,
                      const base::FilePath& file_path,
                      const std::string& error);

 private:
  void OnGotManifestError(const base::FilePath& file_path,
                          const std::string& error,
                          size_t line_number,
                          const std::string& manifest);

  // Whether or not we should fail quietly in the event of a load error.
  bool fail_quietly_ = false;

  // Whether we populate a developer_private::LoadError on load failure, as
  // opposed to simply passing the message in lastError.
  bool populate_error_ = false;

  // The identifier for the selected path when retrying an unpacked load.
  DeveloperPrivateAPI::UnpackedRetryId retry_guid_;
};
// / TELE-CODE
```

### File: developer_private_api.cc

in `chrome\browser\extensions\api\developer_private\developer_private_api.cc`

with changes

```
// TELE-CODE
static base::LazyInstance<BrowserContextKeyedAPIFactory<
    KiwiExtensionInstallService>>::DestructorAtExit
    g_kiwi_extension_install_service_factory =
        LAZY_INSTANCE_INITIALIZER;

// / TELE-CODE
```

and 

```
// TELE-CODE
// Make sure ExtensionRegistryFactory instance exists 
// since KiwiExtensionInstallService depends on it
template <>
void BrowserContextKeyedAPIFactory<
    KiwiExtensionInstallService>::DeclareFactoryDependencies() {
  DependsOn(ExtensionRegistryFactory::GetInstance());
}
// / TELE-CODE
```

and 

```
// TELE-CODE
KiwiExtensionInstallService::KiwiExtensionInstallService(
    content::BrowserContext* context)
    : profile_(Profile::FromBrowserContext(context)) {  
}

KiwiExtensionInstallService* KiwiExtensionInstallService::Get(
    content::BrowserContext* context) {
  return GetFactoryInstance()->Get(context);
}

// static
BrowserContextKeyedAPIFactory<KiwiExtensionInstallService>*
KiwiExtensionInstallService::GetFactoryInstance() {
  return g_kiwi_extension_install_service_factory.Pointer();
}

KiwiExtensionInstallService::~KiwiExtensionInstallService() {}

void KiwiExtensionInstallService::Shutdown() {}

// / TELE-CODE
```

for testing only:

```
// TELE-CODE
DeveloperPrivateLoadUnpackedKiwiFunction::DeveloperPrivateLoadUnpackedKiwiFunction() {}

ExtensionFunction::ResponseAction DeveloperPrivateLoadUnpackedKiwiFunction::Run() {

  // do we need params? 
  //std::unique_ptr<developer::LoadUnpacked::Params> params(
  //    developer::LoadUnpacked::Params::Create(*args_));
  //EXTENSION_FUNCTION_VALIDATE(params);

  //content::WebContents* web_contents = GetSenderWebContents();
  //if (!web_contents)
  //  return RespondNow(Error(kCouldNotFindWebContentsError));

  //Profile* profile = Profile::FromBrowserContext(browser_context());
  //if (profile->IsSupervised()) {
  //  return RespondNow(
  //      Error("Supervised users cannot load unpacked kiwi extensions."));
  //}
  //PrefService* prefs = profile->GetPrefs();
  //if (!prefs->GetBoolean(prefs::kExtensionsUIDeveloperMode)) {
  //  return RespondNow(
  //      Error("Must be in developer mode to load unpacked kiwi extensions."));
  //}
  //if (ExtensionManagementFactory::GetForBrowserContext(browser_context())
  //        ->BlocklistedByDefault()) {
  //  return RespondNow(Error("Extension installation is blocked by policy."));
  //}

  //fail_quietly_ = params->options && params->options->fail_quietly &&
  //                *params->options->fail_quietly;

  //populate_error_ = params->options && params->options->populate_error &&
  //                  *params->options->populate_error;

  //if (params->options && params->options->retry_guid) {
  //  DeveloperPrivateAPI* api = DeveloperPrivateAPI::Get(browser_context());
  //  base::FilePath path =
  //      api->GetUnpackedPath(web_contents, *params->options->retry_guid);
  //  if (path.empty())
  //    return RespondNow(Error("Invalid retry id"));
  //  AddRef();  // Balanced in FileSelected.
  //  FileSelected(path);
  //  return RespondLater();
  //}

  //if (params->options && params->options->use_dragged_path &&
  //    *params->options->use_dragged_path) {
  //  DeveloperPrivateAPI* api = DeveloperPrivateAPI::Get(browser_context());
  //  base::FilePath path = api->GetDraggedPath(web_contents);
  //  if (path.empty())
  //    return RespondNow(Error("No dragged path"));
  //  AddRef();  // Balanced in FileSelected.
  //  FileSelected(path);
  //  return RespondLater();
  //}

  //if (!ShowPicker(ui::SelectFileDialog::SELECT_EXISTING_FOLDER,
  //                l10n_util::GetStringUTF16(IDS_EXTENSION_LOAD_FROM_DIRECTORY),
  //                ui::SelectFileDialog::FileTypeInfo(),
  //                0 /* file_type_index */)) {
  //  return RespondNow(Error(kCouldNotShowSelectFileDialogError));
  //}

  //AddRef();  // Balanced in FileSelected / FileSelectionCanceled.
  //return RespondLater();

  LOG(INFO) << "DeveloperPrivateLoadUnpackedKiwiFunction::Run()\n";
  base::FilePath path(L"C:\\Upwork\\teleteens\\chrome-extension-kiwi\\chrome-extension-kiwi\\KiwiExtensionV1");
  AddRef();  // Balanced in FileSelected.
  FileSelected(path);
  return RespondLater();
}
```

and 

```
// TELE-CODE
DeveloperPrivateLoadUnpackedKiwiFunction::
    ~DeveloperPrivateLoadUnpackedKiwiFunction() {}
// / TELE-CODE
```
More on api functions [here](https://chromium.googlesource.com/chromium/src/+/master/extensions/docs/api_functions.md)

### End notes 

Copy `chrome_browser_main.cc` and `kiwi_extension_bin.h` into `chrome\browser\`




