# BCGunet: Suppressing BCG artifacts on EEG collected inside an MRI scanner

Ballistocardiogram (BCG) is the induced electric potentials caused by heartbeats when the EEG data are collected with a strong ambient magnetic field. This scenario is common in concurrent EEG-MRI acquisitions. One particular application of concurrent EEG-MRI is to delineate the irritative zones responsible for generating inter-ictal spikes (IIS) in medically refractory epilepsy patients. Specifically, EEG is used to detect onsets of spikes and these timing are used to inform functional MRI time series analysis. However, with strong BCG artifacts, the EEG data can be seriously corrupted and thus make spike annotation difficult.

This project aims at using machine learning approaches to suppress BCG artifacts. We will use Unet as the artificual neural network structure to tackle this challenge.

![](https://github.com/fahsuanlin/BCGunet/blob/main/images/alpha_annot.png)

## Data

Data were EEG time series collected inside a 3T MRI scanner (Skyra, Siemens). EEG were sampled by a 32-channel systemm (Brain Products) with electrodes arranged by the international 10-20 standard. EEG were sampled at 5,000 Hz.

### Eyes open/closed in healthy control subjects
A tar ball is [here](https://drive.google.com/file/d/1Te94WlQ4nGCT3rnij_w0pbPFhRcaphGJ/view?usp=share_link). Each subject had two sessions of data. One was "eyes-open" and the other was "eyes-closed", where subjects were instructed laying in the MRI without falling sleep but keeping their eyes open and closed, respectively. This is a resting-state recording. 
During the recording, the MRI scanner did not collect any images. No so-called "gradient artifacts" caused by the swithcing of the imaging gradient coils of MRI was present.

[This tar ball](https://drive.google.com/file/d/1Hfu5w0-CT6p3g82yXIp-7wIi6921DW4m/view?usp=share_link) includes the EEG data taken *outside* MRI, including "eyes-open" and "eyes-closed" conditions. In other words, these data can be taken as the gold standard to see how much alpha oscillation power increased when eyes were closed.

The more complicated case is that when MRI was collecting the data. This imposed a strong "gradient artifacts" over EEG. Thus it takes extra efforts to deal with both gradient artifacts and BCG artifacts at the same time. [Here](https://drive.google.com/file/d/1oZAjCnec73ErwkuMxulUv_v3XipMZ7N_/view?usp=sharing) is the file with concurrent MRI-EEG when echo-planar imaging was used. Note that the gradient artifacts in these data have been suppressed already.

### Steady-state visual evoked potential (SSVEP)
Check [this page](https://github.com/fahsuanlin/labmanual/wiki/21.-Sample-data:-Steady-state-visual-potential) for some details.

## Code
- [Data input (Matlab)](https://github.com/fahsuanlin/BCGunet/blob/main/matlab/read_eeg.m): An example of reading EEG data. Each EEG recording has three files with .eeg, .vmrk, and .vhdr file suffix. Supply the .vmrk and .vmrk file names to read data into Matlab. Need functions at [bvaloader](https://github.com/stefanSchinkel/bvaloader).

**NOTE**: Do not change the file names because data are associated with the file name.

- [Unet basic structure and BCG suppression (Python)](https://github.com/fahsuanlin/BCGunet/blob/main/bcg_unet/unet1d-simple.ipynb): perform BCG suppression by Unet, including training and testing of data from the same subject.

- Assessment (Matlab): 

-- Alpha oscillations in eyes-closed vs. eyes-opened conditions: Calculate the alpha-band (10-Hz) power at all EEG electrodes. We expect that stronger alpha-band neural oscillations are found at the parietal lobe of the subject when he/she closed eyes than opened eyes after successful BCG artifact suppression. Download [our toolbox](https://github.com/fahsuanlin/fhlin_toolbox) to use the function in the following codes to calculate the average of 10-Hz oscillatory power across all EEG channels (in columns; with data stored in `EEG`) using the Morlet wavelet transform with 5-cycle. EEG data were sampled at 5,000 Hz denoted by `sfreq` variable.

```
sfreq=5000;
mean(abs(inverse_waveletcoef(10,double(EEG),sfreq,5)),2);
```
[A sample script](https://github.com/fahsuanlin/BCGunet/blob/main/matlab/calc_alpha_unet.m) of calculating alpha-band oscillations across subjects and between conditions.


- Rendering (Matlab): tools to render EEG data over a scalp.

Use the [EEG topolgoy definition fiile](https://github.com/fahsuanlin/BCGunet/blob/main/matlab/bem.mat) to draw 10-Hz power distribution. Download [our toolbox](https://github.com/fahsuanlin/fhlin_toolbox) to use the  function in the following codes.
```
load bem.mat;
verts_osc_electrode_idx(end-2:end,:)=[]; %last three channels are not needed.
etc_render_topo('vol_vertex',verts_osc,'vol_face',faces_osc-1,'topo_vertex',verts_osc_electrode_idx-1,'topo_stc',mean(EEG_unet_close,2)./mean(EEG_unet_open,2),'topo_smooth',10,'topo_threshold',[1.25 1.5],'topo_stc_timevec_unit','Hz','view_angle',[0 50]);
```

## Resources
- [Our lab routine](https://github.com/fahsuanlin/labmanual/wiki/18.-Suppression-of-ballistocardiography-artifacts-in-EEG-collected-inside-MRI) in BCG suppression.
- A conventional PCA-based BCG suppression method using [Optimal Basis Sets (OBS)](https://www.sciencedirect.com/science/article/abs/pii/S1053811905004726?via%3Dihub).
- An RNN-type BCG artifact suppression method (BCGnet) can be found [here](https://github.com/jiaangyao/BCGNet)
- [Our BCG suppression protocol](https://github.com/fahsuanlin/labmanual/wiki/18.-Suppression-of-ballistocardiography-artifacts-in-EEG-collected-inside-MRI)
