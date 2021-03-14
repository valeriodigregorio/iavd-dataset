# Imperial Assault Vassal Deck File Dataset (iavd-dataset)
This library contains a dataset that can be used to read and write Imperial Assault Vassal Deck (IAVD) Files.

# Compatibility
Current dataset supports:

* Vassal v3.5.0 (http://www.vassalengine.org/)
* Imperial Assault Skirmish module v12.5.1 (http://www.vassalengine.org/wiki/Module:Star_Wars:_Imperial_Assault)
* IACP v5.1 (https://ia-continuityproject.com/)

# Deck save file format
A deck save file for the Imperial Assault Skirmish module for Vassal is a concatenation of strings. Such a file starts with the string "DECK\t" and follows with a concatenation of multiple strings. Each concatenated string contains a marshalled version of one card. The marshalled version of a card always starts with the "ESC" (ASCII 27) character.

```
header = "_DECK\t"
separator = char(27)
card[i] = separator + marshalled_card[i]
deck_save_file = header + card[1] + ... + card[N]
```

iavd-dataset provides a set of files that we will call "Imperial Assault Vassal Deck" (IAVD) files. Each IAVD file is a deck save file with only one card from the Imperial Assault Skirmish module for Vassal. This is equivalent to:

```iavd_file[i] = header + separator + card[i]```

Users of iavd-dataset can concatenate multiple IAVD files to obtain a well-formed deck save file for the Imperial Assault Skirmish module for Vassal. Following pseudo-code shows how:

```
header = "DECK\t"
deck_save_file.write(header)
for i in cards_in_my_army:
  card = iavd_file[i].strip(header)
  deck_save_file.write(card)
```

IAVD files are available in following folders:

* ```./ffg``` contains cards for Imperial Assault FFG card system
* ```./iacp``` contains cards for Imperial Assault Continuity Project card system

Each IAVD file is named with its respective Vassal card ID.

# JSON metadata
Bundled with this dataset there is a set of JSON files that provides all the metadata required to read a deck save file for the Imperial Assault Skirmish module for Vassal. Available JSON files are:

* ```./ffg_deployment_dataset.json``` contains description for all the deployment cards for Imperial Assault FFG card system
* ```./iacp_deployment_dataset.json``` contains description for all the deployment cards for Imperial Assault Continuity Project card system
* ```./ffg_command_dataset.json``` contains description for all the command cards for Imperial Assault FFG card system
* ```./iacp_command_dataset.json``` contains description for all the command cards for Imperial Assault Continuity Project card system

When reading a deck save file for the Imperial Assault Skirmish module for Vassal, the user can strip off the "DECK\t" header, then look for the "ESC" (ASCII 27) character. Text between consecutive "ESC" (ASCII 27) characters (or between "ESC" (ASCII 27) character and End of File) must match the following regular expression:

```IAVD_REGEX = ".*piece;;;.*?;(.*?)/.*\\	(-*1)\\.*\\	null;\d*;\d*;(\d*)"```

Such regex will match two groups:

1. First group represents the Vassal card name. This name does not match the official name used by FFG or IACP. For this reason we suggest to use the JSON metadata for a more reliable description.
2. Second group represents the card system. Value -1 stands for FFG and value 1 stands for IACP, however if the Vassal card ID can refer an IACP card (in other words a card that does not exist in FFG card system) this value must be ignored.
3. Third group represents the Vassal card ID.

Unfortunately, due to Vassal limitations, lookup of Vassal card IDs in JSON metadata files is not straightforward. One way to implement the lookup is described in the following pseudo-code:

```
metadata[FFG][Deployment] = load_and_parse_json_file("./ffg_deployment_dataset.json")
metadata[FFG][Command] = load_and_parse_json_file("./ffg_command_dataset.json")
metadata[IACP][Deployment] = load_and_parse_json_file("./iacp_deployment_dataset.json")
metadata[IACP][Command] = load_and_parse_json_file("./iacp_command_dataset.json")

deck = read_text_file(path_to_deck_save_file)
deck = deck.strip("DECK\t")
cards = deck.split(char(27))

for card in cards:
  name, sys, id = apply_regex(IAVD_REGEX, card)
  if metadata[FFG][Deployment].contains(id) and sys == -1:
    load_card_into_army(metadata[FFG][Deployment].get(id))
  elif metadata[FFG][Command].contains(id) and sys == -1:
    load_card_into_army(metadata[FFG][Command].get(id))
  elif metadata[IACP][Deployment].contains(id):
    load_card_into_army(metadata[IACP][Deployment].get(id))
  elif metadata[IACP][Command].contains(id):
    load_card_into_army(metadata[IACP][Command].get(id))
  else:
    error()
```

# Caveats

* Vassal card IDs for deployment cards and command cards are different. If a Vassal card ID is not present inside the JSON metadata file for deployment cards, the user should look into the JSON metadata file for command cards.
* Different Vassal card IDs can refer the same command card. In example "Adrenaline" command card has both ID 8983 and 10556. This is due to the card being listed both in "Wookiee" and "Any Figure" sections in the Imperial Assault Skirmish module for Vassal. When users write card "Adrenaline" in a deck save file, they can use either IAVD file 8983 or 10556 because there is no difference.
* Same Vassal card ID may be valid both in FFG card system and in IACP card system. In example card with ID 8569 refers to "Heroic Effort" both in ```./ffg_deployment_dataset.json``` and ```./iacp_deployment_dataset.json```. However ```./iacp/8569``` does not exists and ```./iacp_deployment_dataset.json``` refers to ```./ffg/8569``` because the card has the same image both in FFG and IACP case. Another example is card with ID 547. This card refers to "Biv Bodhrik" both in ```./ffg_deployment_dataset.json``` and ```./iacp_deployment_dataset.json```. However ```./ffg/547``` and ```./iacp/547``` both exist and they appear in Vassal with different images depending on the card system.
