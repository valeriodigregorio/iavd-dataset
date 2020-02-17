# Imperial Assault Vassal Deck File Dataset (iavd-dataset)
This library contains a dataset that can be used to read and write Imperial Assault Vassal Deck (IAVD) Files.

# Compatibility
Current dataset supports:

* Vassal v3.2.17 (http://www.vassalengine.org/)
* Imperial Assault Skirmish module v12.3.1 (http://www.vassalengine.org/wiki/Module:Star_Wars:_Imperial_Assault)
* IACP v3.0 (https://ia-continuityproject.com/)

## Folder structure
Files are split in folders:

* _ffg_ contains cards for Imperial Assault vanilla
* _iacp_ contains cards for Imperial Assault Continuity Project

## IAVD File format
An Imperial Assault Vassal Deck File is a concatenation of strings. Such file starts with string "DECK\t" and follows with a concatenation of string each one containing a marshalled version of a card.
This dataset is a set of files containing the marshalled versions of all card cards available in Vassal. Each file is named with a number representing the Vassal ID of the card.

## Imperial Assault Unique Naming (IAUN) format
We defined an unique naming format that can be universally used to refer to Imperial Assault cards. Unique naming format is defined as follows:

_affiliation_ _name_, [_description_|Elite|Regular] [FFG|IACP]

_Affiliation_ can be one of:

* Rebel
* Imperial
* Mercenary
* Neutral

_Name_ is the name of the card as reported by the RRG. _Description_ is present only if the card is a unique card. If card is not unique, format will report the type of the card:

* Elite
* Regular

The card system the card belongs is used to terminate the IAUN format and can be one of:

* FFG
* IACP

## Mappings
Two mappings are provided with the current dataset:

* _iaun_to_vassal.json_ that maps Imperial Assault Unique Names to Vassal IDs 
* _vassal_to_iaun.json_ that maps Vassal IDs to Imperial Assault Unique Names
