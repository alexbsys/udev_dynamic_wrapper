# udev_dynamic_wrapper
Wrapper for libudev functions which loads libudev.so dynamically in runtime
Dependency-injection based module which loads libudev completely dynamically with dlopen/dlsym, without necessary for link dependency to app or library or include <libudev.h>

## How to build
```
mkdir build
cd build
cmake ..
make -j
```
Small library and example will be generated.
For link wrapper to your project, just add -ludev_dynamic_wrapper.

## How to use
Just replace your calls to udev_xxx functions to wrapper: udev_mod->udev_new(), udev_mod->udev_device_new_from_devnum()


```
#include "dynamic_modules_loader.h"
#include "udev_module_provider.h"

std::string get_hdd_serial_number(std::shared_ptr<IUdevModuleProvider> udev_mod, const std::string& disk_dev_path) {
  std::string serial_number;
  struct IUdevModuleProvider::udev *ud      = NULL;
  struct stat statbuf;
  struct IUdevModuleProvider::udev_device *device  = NULL;
  struct IUdevModuleProvider::udev_list_entry *entry   = NULL;

  ud = udev_mod->udev_new();
  if (NULL == ud) 
    return serial_number;

  if (0 != stat(disk_dev_path.c_str(), &statbuf))
    return serial_number;

  device = udev_mod->udev_device_new_from_devnum(ud, 'b', statbuf.st_rdev);
  if (NULL == device) 
    return serial_number;
      
  entry = udev_mod->udev_device_get_properties_list_entry(device);
  while (NULL != entry) {
    if (0 == strcmp(udev_mod->udev_list_entry_get_name(entry),"ID_SERIAL")) 
      break;

    entry = udev_mod->udev_list_entry_get_next(entry);
  }

  serial_number = udev_mod->udev_list_entry_get_value(entry);

  udev_mod->udev_device_unref(device);
  udev_mod->udev_unref(ud);

  return serial_number;
}


```
