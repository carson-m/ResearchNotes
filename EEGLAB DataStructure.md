# EEGLAB DataStructure

- EEG: the current EEG dataset
- ALLEEG: array of all loaded EEG datasets
- CURRENTSET: the index of the current dataset
- LASTCOM: the last command issued from the EEGLAB menu
- ALLCOM: all the commands issued from the EEGLAB menu
- STUDY: the EEGLAB group analysis structure
- CURRENTSTUDY: 1 if EEGLAB performing group analysis, 0 otherwise

## EEG

EEGLAB variable *EEG* is a MATLAB structure that contains all the information about the current EEGLAB dataset.

### chanlocs

This EEG-structure field stores information about the EEG channel locations and channel names.

### event

The *EEG* structure field contains records of the experimental events that occurred while the data was being recorded, plus possible additional user-defined events.

### urevent

Holds all the event information that was originally loaded into the dataset plus events that were manually added by the user. When continuous data is first loaded, the content of this structure is identical to contents of the *EEG.event* structure (minus the *urevent* pointer field of *EEG.event*). However, as users remove events from the dataset through artifact rejection or extract epochs from the data, some of the original (ur) events continue to exist only in the urevent structure.

## ALLEEG

EEGLAB variable *ALLEEG* is a MATLAB array that holds all the datasets in the current EEGLAB/MATLAB workspace. In fact *ALLEEG* is a structure array of *EEG* datasets (described above). If, in the current EEGLAB session you have one datasets loaded, *ALLEEG* will be equal to *EEG*.