---
title: Some Flutter Programming Rules with Supa
date: 2025-02-16 00:01:00 +0700
categories: [Mobile Development, Flutter]
tags: [Mobile Development, Flutter]
pin: false
---

## Introduction

### Flutter and its main advantages

- Flutter provides a comprehensive set of tools, designs, and widgets to easily and quickly create interfaces.
- Available widgets and layouts can be customized to create interfaces.
- Excellent theme customization capabilities, available and consistent across the entire application.

## Application Structure

The application structure is as follows:

```
- blocs: contains shared bloc logic
- config
  - get_it.dart: configuration for getIt, used for dependency injection
  - supa_subsystem.dart: configuration for the list of subsystems in the system
  - supa_locale.dart: configuration for supported languages
- extensions: extends existing classes (only includes utilities, no business logic)
- modules: each subdirectory is the code for a subsystem, independent, can be cloned into a separate app
- notifications: contains notification interfaces and handlers for specific notifications
- router: contains go_router configurations, defines routing for the entire app, with separate routing files for each subsystem
- services: contains shared service logic: location, media, etc.
- themes: contains theme configurations for the application, easily customize the interface with themes here, aiming for multi-theme functionality
- widgets: contains shared widgets
```

## Localization

Due to the inconvenience of Flutter's built-in localization tools, Supa developed its own library with a unique localization management mechanism called `supa_l10n_manager` as follows:

### Guidelines and Directory Structure

Localization files are stored in a flat JSON format with the following rule: `subsystem.namespace.key1.key2`

Where:

- subsystem is the name of the subsystem. For example: `spend`, `work`, `general`, ...
- namespace can be the name of an entity or screen to group keys together. For example: `home`, `user`, `product`, ...
- `key1`, `key2`: Supports up to 2 levels of subkeys (However, key2 should be limited)
- Since JSON has a flat structure, avoid overwriting between 3-level and 4-level keys with the same prefix. For example, if there is a key `work.home.inspection`, then keys like `work.home.inspection.noChildKeyHere` are not allowed as subkeys will be overwritten during source code scanning, leading to runtime translation errors.

### Source Code Scanning for Key Generation

The package supports a command to scan the entire source code (Dart files) to generate corresponding JSON files without manual work.

Users need to "mark" the generated keys when writing code using the following syntax: `translate('subsystem.namespace.key1.key2')`

Since the package provides a simple syntax scanning and recognition mechanism, it is mandatory to mark translation keys by directly calling `translate`. Otherwise, the package will not recognize them.

Incorrect example:

```dart
final key = 'subsystem.namespace.key1.key2';
final translatedText = translate(key); /// Key is not marked correctly
```

Correct example:

```dart
final translatedText = translate('subsystem.namespace.key1.key2');
```

Note: if there are 2 levels to translate, write them as two statements like this:

```dart
final translatedDateTime = translate('system.namespace.formattedDate', {'date': formattedDate});
final translatedWarningText = translate('system.namespace.deadlineExceededWarning', {'deadline': translatedDateTime});
```

To scan and generate JSON files, use the following command:

```sh
dart run supa_l10n_manager extract --locale vi
```

Where: you can replace `vi` with the corresponding locale code of the desired language.

### Merge Functionality

For each subsystem, the package will generate a corresponding JSON file. This ensures the file is small and independent between subsystems, making it easy to handle conflicts on Git and divide work within the team during development.

The application will use a single JSON file for each language, so it is necessary to "merge" the files together. To merge files within the same language (locale), use the following command:

```sh
dart run supa_l10n_manager merge
```

### Quick Edit Tool

Supa provides a Chrome extension for quick editing of JSON files with the following features:

- Parallel translation of languages on the web interface
- AI-assisted automatic translation for new languages
- Export keys from Figma design (with node naming rules)

Download the extension: [Localization Editor on Chrome Web Store](https://chromewebstore.google.com/detail/localization-editor/eepapdeoidmlaihgncfaphiccekeghjn?hl=vi)

### Parameter Support

`supa_l10n_manager` supports passing parameters when translating from a translation key to a display label for each language.

Syntax:

```dart
translate('subsystem.namespace.key1[.key2]', {
  'param1': value1,
  'param2': value2,
});
```

Note:
- Parameters are passed as a map
- Values must be simple variable names, avoid special names, method/function calls, multi-level property access.

Correct example:

```dart
translate('work.taskAssignment.taskAssignmentCount', {
  'count': count,
});
```

Incorrect/should be avoided:

```dart
translate('work.taskAssignment.taskAssignmentCount', {
  'count': widget.taskAssignment.list.countLength(), /// Multiple-level property access, method call, etc.
});
```

### Limitations:

Currently, there are some limitations:
- Does not support plural forms, verb conjugation in some languages
- Translation keys are strings and created arbitrarily, so they must be entered according to rules to avoid input errors, typos, duplicates

## Interface (Theming)

Flutter Material supports app structure and most widgets have a stable structure to quickly build application layouts.

Developers only need to:

- Adjust the theme to match the design
- Follow atomic design, create customizable and reusable widgets to stabilize the interface and increase maintainability

### Text theme

Flutter has predefined text themes for all text used in the app:

| Type     | Usage Location                         |
|----------|----------------------------------------|
| Title    | Main page title, header                |
| Headline | Subtitle, section title                |
| Body     | Main content, text paragraph           |
| Label    | Text in button, input label            |
| Display  | Highlight text, decoration (hero text) |

- Each style provides 3 sizes: `small`, `medium`, `large`

- In case of customization: use the `.copyWith` method to customize. Note: only use for font size and color, very limited use for other properties.

- Line height, letter spacing properties are predefined and fixed according to the text theme, do not customize.

### Spacing Guidelines

#### General Convention

- `margin` and `padding` are multiples of 4, in some special cases, 2 (half of 4), 6 (half of 12) can be used

- Normal icon size for buttons is 24 with 8 padding on each side, total size of icon button is 40x40

- Follow Figma design but be flexible: objects that resize according to content (ListView, Text) should use Flexbox with Expanded, Flexible to balance, avoid overflow due to different mobile screen sizes.

#### Spacing for Objects

- Use vertical `margin` for ListItem. Split each side. For example: if Figma design has spacing between 2 ListItems as 16, then each margin side will be 8

- Use horizontal `padding` for ListView.

- If there is a spacing of 16 between 2 ListItems, since each side has a margin of 8, the first and last elements will only be 8 away from the top and bottom edges. Add an additional 8 padding (half spacing) for the outer ListView container.

- Container has a common `padding` of 16, or 12 or 8 in some cases depending on the design, but needs to be consistent to avoid inconsistency

- Container has `BorderRadius` of 4, with `elevation` of 1 and shadow according to the border color `Theme.of(context).colorScheme.outline`

### Color scheme:

Flutter provides a color scheme in the theme `Theme.of(context).colorScheme` with the following values:

| Key            | Function                                                                          |
|----------------|-----------------------------------------------------------------------------------|
| primary        | Main color of the app                                                             |
| secondary      | Secondary color, in less strong buttons                                           |
| tertiary       | Third secondary color, not clearly defined how to apply                           |
| error          | Error notification color, usually red but with a hex code depending on the design |
| surface        | Background color                                                                  |
| outline        | Border, shadow color                                                              |
| outlineVariant | Lighter border, shadow color (blurred)                                            |

Additionally, Supa extends with the following getters:

| Getter  | Function                                      |
|---------|-----------------------------------------------|
| success | Green color, used for success/valid cases     |
| info    | Light blue color, used to display information |
| warning | Orange color, used to display warnings        |

Each color above will have a corresponding text color with the prefix `on`. For example: `onPrimary`, `onSurface`

Note: always use the theme, do not hard code colors

## Some Rules, Syntax, and Structure

### Translation

- Use clear, concise English names
- For level 3 and above (after subsystem and namespace names), a short English sentence can be used to convey meaning, also supporting quick AI translation. THIS IS IMPORTANT DUE TO LIMITED RESOURCES.

Example: `work.error.anErrorOccurredDuringProcess`: An error occurred during processing. easy to translate, the key is understandable

- DO NOT use ambiguous words, unclear numbering. For example: `tab1`, `tab2`. Tabs need names.

### Writing Widgets

- Follow atomic design, design widgets by level for easy reuse, MORE IMPORTANTLY for CONSISTENCY, and EASY TO MODIFY

- Prioritize quick product development but need:
  + Stable layout
  + Clear structure, do not cram into one file.

- Can use ChatGPT to refactor after coding.
