"contributes": {
  "configuration": {
    "title": "Static Text Scanner",
    "properties": {
      "staticTextScanner.languages": {
        "type": "array",
        "description": "List of languages and their i18n file paths.",
        "items": {
          "type": "object",
          "properties": {
            "language": { "type": "string" },
            "path": { "type": "string" }
          }
        },
        "default": []
      }
    }
  }
}


vscode.commands.executeCommand('workbench.action.openSettings', '@ext:yourPublisher.yourExtension');
