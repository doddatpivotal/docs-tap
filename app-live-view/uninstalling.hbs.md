# Uninstall Application Live View

This topic tells you how to uninstall Application Live View from Tanzu Application Platform
(commonly known as TAP).

To uninstall the Application Live View back end, Application Live View connector, and
Application Live View convention server, run:

```console
tanzu package installed delete appliveview -n tap-install
tanzu package installed delete appliveview-connector -n tap-install
tanzu package installed delete appliveview-conventions -n tap-install
```
