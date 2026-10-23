# Drive label evidence

The owner supplied these two photos and authorized their publication.
The copies keep the original image bytes.

## Method

RapidOCR extracted text from both JPEG files on 2026-10-22.
The raw output and JSON method records are stored in this folder.
The raw output has OCR confidence, not exact-text confidence.
The README facts were also compared visually with the supplied photos.

## Visually confirmed markings

| Photo | Printed field | Reading |
|---|---|---|
| [Manufacturer label](../images/manufacturer-label.jpg) | Manufacturer | Hitachi-LG Data Storage, Inc. |
| Manufacturer label | Supply | 5 V DC |
| Manufacturer label | Label code | MEZ650643 |
| [Model label](../images/model-label.jpg) | Product description | UHD Blu-ray / DVD Writer |
| Model label | MODEL | BU50N |
| Model label | Model code | ANCK1N0 |
| Model label | F/W | CC04 |
| Model label | H/W | A |
| Model label | MECHA | A |
| Model label | MANUFACTURED | JANUARY 2022 |
| Model label | Numeric manufacture date | 2022 01 08 |
| Model label | FACTORY ID | GH |
| Model label | Country | MADE IN CHINA |
| Model label | Regulatory identifier | MSIP-REM-HLD-BU40N |

OCR read the firmware as `CCO4` and the regulatory model as `BU4ON`.
Visual comparison establishes `CC04` and `BU40N`, with zeroes.
The raw OCR files keep the original recognition output.

## What the evidence establishes

The photographed unit has both BU50N and BU40N text in different fields.
BU50N is the model field.
BU40N is part of the regulatory identifier.
The photos do not establish that every unit from the same Amazon listing is identical.

The owner reports no separate FRU label on this unit.
FRU means field-replaceable unit, a service replacement part.
The photos do not show a separate FRU part-number label.
No other unit's FRU status was examined.

## Records

- [Manufacturer OCR](manufacturer-label.txt) and [method record](manufacturer-label.txt.json).
- [Model OCR](model-label.txt) and [method record](model-label.txt.json).

The drive platform, USB bridge and successful firmware result come from software reads.
They are separate from the physical-label transcription.
