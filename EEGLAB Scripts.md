# EEGLAB Scripts

## eeglab

init

```matlab
[ALLEEG EEG CURRENTSET ALLCOM] = eeglab;
```

Refresh GUI

```matlab
eeglab redraw
```

## history

view the session history for the current EEGLAB session

```matlab
eegh
```



## eeg_store

Create a new dataset, stores the dataset in the *ALLEEG* structure

```matlab
[ALLEEG EEG CURRENTSET] = eeg_store(ALLEEG, EEG);
```

Overwrite a current dataset, stores the dataset in the *ALLEEG* structure

```matlab
[ALLEEG EEG CURRENTSET] = eeg_store(ALLEEG, EEG, CURRENTSET);
```

Create a new dataset and set the new dataset to be dataset number 2. If 2 already exists, it will be overwritten.

```matlab
[ALLEEG EEG] = eeg_store(ALLEEG, EEG, 2);
```

Note: *EEG* contains only the current dataset.

## pop_newset

To modify the current dataset with its accumulated changes

```matlab
[ALLEEG EEG CURRENTSET] = pop_newset(ALLEEG, EEG, CURRENTSET,'overwrite', 'on');
```

If you wish to create a new dataset to hold the modified structure

```matlab
[ALLEEG EEG CURRENTSET] = pop_newset(ALLEEG, EEG, CURRENTSET);
```

## eeg_checkset

Check the internal consistency of the modified dataset.

```matlab
EEG = eeg_checkset(EEG);
```

Check the internal consistency of the modified dataset. runs extra checks for event consistency (possibly taking some time to complete) and regenerates the *EEG.epoch* structures from the *EEG.event* information. This command is only used when the event structure is being altered.

```matlab
EEG = eeg_checkset(EEG, 'eventconsistency');
```

## pop_loadset

Old call method.

```matlab
EEG = pop_loadset( 'eeglab_data.set', '/matlab/eeglab/sample_data');
```

Load under options.

```
EEGOUT = pop_loadset( 'key1', 'val1', 'key2', 'val2', ...);
```

```matlab
%  Optional inputs:
%    'filename'  - [string] dataset filename. Default pops up a graphical
%                  interface to browse for a data file.
%    'filepath'  - [string] dataset filepath. Default is current folder. 
%    'loadmode'  - ['all', 'info', integer] 'all' -> load the data and
%                  the dataset structure. 'info' -> load only the dataset 
%                  structure but not the actual data. [integer] ->  load only 
%                  a specific channel. This is efficient when data is stored 
%                  in a separate '.dat' file in which individual channels 
%                  may be loaded independently of each other. {default: 'all'}
%    'eeg'       - [EEG structure] reload current dataset
```

Pop up window to input arguments

```matlab
EEGOUT = pop_loadset;
```



## Consistency

To maintain consistency between the two main EEGLAB structures (*EEG* and *ALLEEG*), you need to update the *ALLEEG* every time you modify the *EEG* structure.

## pop_editoptions

Change option to process multiple datasets.

```matlab
pop_editoptions( 'option_storedisk', 0);
```