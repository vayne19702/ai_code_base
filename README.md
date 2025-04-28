"contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "staticTextScannerContainer",
          "title": "Scanner",
          "icon": "media/icon.svg"
        }
      ]
    },
    "views": {
      "staticTextScannerContainer": [
        {
          "icon": "media/icon.svg",
          "id": "staticTextScanner",
          "name": "Static Text Scanner"
        }
      ]
    },
    "commands": [
      {
        "command": "staticTextScanner.scan",
        "title": "Scan Static Text",
        "icon": "$(play)"
      }
    ],
    "menus": {
      "view/title": [
        {
          "command": "staticTextScanner.scan",
          "when": "view == staticTextScanner",
          "group": "navigation"
        }
      ]
    }
  },
