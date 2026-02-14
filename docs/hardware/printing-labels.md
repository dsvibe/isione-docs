# Printing Labels

Guide for creating and 3D printing enclosure slot labels.

## Generating Label STL Files

Use the [Label STL Generator](https://dsvibe.github.io/isione-label-stl-generator) web tool to create label files:

1. Open the label generator
2. Search and select a Lucide icon
3. Enter the task name
4. Download the generated STL

The tool combines the base label geometry with your icon and text, ready for slicing.

## Print Settings

- **Printer**: Prusa MK4S
- **Slicer**: PrusaSlicer (auto-detects colour change point)
- **Profile**: Default settings (still experimenting)

## Filament

| Filament | Colour |
|----------|--------|
| [Prusament PLA Prusa Galaxy Black](https://www.prusa3d.com/de/produkt/prusament-pla-prusa-galaxy-black-2kg-nfc/) | Black |
| [DasFilament PLA Tonweiß matt](https://www.dasfilament.de/filament-refill/pla-1-75-mm/801/pla-filament-1-75-mm-tonweiss-matt-1-kg-refill?c=54) | White |

### Colour Combinations

- **Enclosure slot labels**: White text/icon on black base
- **NFC tag labels**: Black icon on white base
