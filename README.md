# CodeEasy Themes - Visual Studio Code Theme Extension

<img src="https://github.com/Parvsharma04/codeeasy/blob/master/logo.png?raw=true" width="100px"/>

## Overview

Welcome to CodeEasy Themes, a collection of visually pleasing and easy-to-read themes for Visual Studio Code. This extension provides a variety of carefully crafted color schemes to enhance your coding experience and make your code more readable.

## Installation

1. Launch Visual Studio Code.
2. Go to the Extensions view by clicking on the Extensions icon in the Activity Bar on the side of the window or use the shortcut `Ctrl+Shift+X`.
3. Search for "CodeEasy Themes".
4. Click on the "Install" button for the CodeEasy Themes extension.

## Available Themes

- **Codeeasy:** A dark theme that reduces eye strain with carefully selected color combinations.
  
   More to come in the future! 

## Activation

1. After installing the extension, open the Command Palette (`Ctrl+Shift+P`).
2. Type `Preferences: Color Theme` and press Enter.
3. Choose your preferred CodeEasy theme from the list.

## Features

- **Readability:** CodeEasy Themes are designed with a focus on readability to make your code more accessible.
- **Syntax Highlighting:** Enjoy carefully chosen colors for syntax elements to improve code comprehension.
- **Consistency:** All themes maintain a consistent color palette for a cohesive coding experience.

## Customization

Feel free to customize the themes according to your preferences. You can easily modify colors using the built-in Visual Studio Code color customization features.

1. Open the Command Palette (`Ctrl+Shift+P`).
2. Type `Preferences: Open Settings (JSON)` and press Enter.
3. Add or modify color settings under the `"editor.tokenColorCustomizations"` section.

```json
"editor.tokenColorCustomizations": {
    "textMateRules": [
        {
            "scope": [
                "comment",
                "keyword"
            ],
            "settings": {
                "foreground": "#008000"
            }
        }
    ]
}
