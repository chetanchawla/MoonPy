![MoonPy_logo](https://github.com/alexteachey/MoonPy/blob/master/moonpy_logo_fixed.png)



# Welcome to MoonPy!
last (non-exhaustive) update: April 29, 2021. Developed by [Alex Teachey](http://www.alexteachey.com).

*Check out the [companion blog](https://moonpy.home.blog/) for a detailed description of changes as they roll out.* (not really managing this anymore).

This document will walk you through the basics of using the MoonPy code. 
MoonPy is designed to make downloading, plotting, detrending, and fitting light curves a breeze, and simply would not exist if not for a number of incredibly powerful packages that have been developed by other scientists (more on that below). Thank you to all those developers! And if you use MoonPy in your research please remember to cite/acknowledge the authors of the packages that have been utilized here. (We'd appreciate a shoutout, too).

MoonPy is kind of like a streamlined astro package aggregator, but in time it will hopefully also contain brand new functionality. 
In particular, support for modeling exomoon transits is coming! (Hence the name, MoonPy). It's not quite ready for prime time, so stay tuned!

## Installation
The easiest way to use MoonPy is to clone the directory, and then create a .pth file that will link this directory such that it will be included in your python path for importing. I've always found this terribly confusing, personally; the best instructions are in [the accepted answer to this question on StackOverflow](https://stackoverflow.com/questions/37006114/anaconda-permanently-include-external-packages-like-in-pythonpath). Basically, create a text file called ```moonpy.pth```, containing the path to MoonPy (wherever you've saved it), and place it in ```site-packages``` directory of the python you're using. 

For example: Start by identifying the path to your python libraries by typing ```which python``` in the terminal. On my machine, for example, this returns ```/opt/anaconda3/bin/python```. With this information you figure out where to place the ```moonpy.pth``` file. Using my path as an example, you will want to place ```moonpy.pth``` within ```/opt/anaconda3/lib/python3.7/site-packages```. Your path may well be different. If you use anaconda environments, it may be something like ```~/anaconda3/envs/{NAME_OF_ENVIRONMENT}/lib/pythonX.X/site-packages/```. 

Once you've done this you should be able to use ```from moonpy import *``` from any working directory.


## Prerequisites

This code requires a number of packages, including: 

* [PyMultiNest](https://johannesbuchner.github.io/PyMultiNest/) - standard parameter estimator and Bayesian evidence calculator, developed by Johannes Buchner as a wrapper to the standard MultiNest code (Feroz and Hobson)
* [emcee](http://dfm.io/emcee/current/) -- alternative to MultiNest, for parameter estimation, developed by Dan Foreman-Mackey (DFM) et al
* [batman](https://www.cfa.harvard.edu/~lkreidberg/batman/) - Standard transit-modeling code by L. Kreidberg
* [corner](https://github.com/dfm/corner.py) -- for visualizing fit parameters, developed by DFM
* [george](https://george.readthedocs.io/en/latest/) -- a Gaussian process regression tool developed by (you guessed it) DFM
* [untrendy](https://github.com/dfm/untrendy) -- another DFM package for fast and easy detrending (currently having difficulties here)

As well as several standard packages (numpy, scipy, astropy, matplotlib, pandas, etc) that likely came with your python distribution.

Package imports that are not standard and are not included in the moonpy package (i.e. the packages above) should all be imported within the relevant function that utilizes them. Therefore, you can hopefully boot up moonpy and use it even if you lack some of the above distributions.


## OVERVIEW
Below you will find the details of some of these tools. At present the user will want to utilize the following functionality:

* **Initialize a light curve object** using *MoonpyLC()*
* **plot the light curve** using *plot_lc()*.
* **generate a Lomb-Scargle** periodogram using *genLS()*.
* **Detrend the light curve** using *detrend()*
* **fit a transit model** to the light curve using *fit()*.
* **plot a transit model from fiducial parameters** taken from the NASA Exoplanet Archive.
* **plot the best fitting model** over the light curve using *plot_bestmodel()*.
* **generate a corner plot** of your the parameters from your fit using *plot_corner()*.
* **get target neighbor attributes** using *get_neighbors()*.
* **identify possible TTVs** using *find_TTVs()*.

Most of the above take keyword arguments that are described below. 



## INITIALIZE A LIGHT CURVE OBJECT.

The core functionality of MoonPy revolves around generating and manipulating a **light curve object**. A large number of attributes are either initially associated with this object, or can be generated after calling functions for the object.

Proper usage is:

*>>> lc_object = MoonpyLC(targetID=None, target_type=None, lc_times=None, lc_fluxes=None, lc_errors=None, lc_flags=None, lc_quarters=None, usr_dict=None, mask_multiple=5, telescope=None, RA=None, Dec=None, coord_format='degrees', search_radius=5, lc_format='pdc', remove_flagged='y', short_cadence=False, ffi='n', save_lc='y', load_lc='n', download='y', is_neighbor='n', attributes_only='n', clobber=None)*


### KEYWORDS

*targetID*: may be a Kepler planet, a KIC star, a KOI, a K2 target, or a planet observed by TESS. If you leave off the prefix you must specify the telescope. Example: "Kepler-1625b", KOI-5084.01, or KIC 4760478. Variations on these (with and without prefix) are (hopefully!) handled to your satisfaction.

*target_type*: May be "kic", "koi", "planet", "toi", "tic", or "epic". The code will attempt to intuit this.

*lc_times*: a time series can be provided, if desired, to utilize MoonPy functionality without downloading from a catalog. Format should either be a single array, or an array of arrays (one for each quarter / segment).

*lc_fluxes*: must have same shape as *lc_times*.

*lc_errors*: Must have same shape as *lc_times*.

*lc_flags*: Must have same shape as *lc_times*.

*lc_quarters*: acceptable values are "all" or an array of quarters. Should have same shape as *lc_times* if there is more than one quarter / segment.

*usr_dict*: a dictionary of object attributes that can be provided for a user-provided lightcurve.

*mask_multiple*: how many times the transit duration on either side of transit midtime that is masked for detrending.

*telescope*: may be "Kepler" / "kepler", "K2" / "k2", "TESS" / "Tess" / "tess", or 'user'.

*RA*: sexagesimal of the form 12h34m14.5s or decimal form.

*Dec*: sexagesimal of the form +50d38m14.25s or decimal form.

*coord_format*: format of your supplied coordinates. An attempt is made to intuit this.

*search_radius*: in arcseconds, the size of the search cone when supplying coordinates.

*lc_format*: May be "sap" (simple aperture photometry) or "pdc" (pre-search data conditioning). Default is "pdc".

*remove_flagged*: automatically removes data points that have quality flags != 0.

*ffi*: can be 'y' or 'n', indicates whether you want to download a full-frame image light curve (probably from TESS).

*save_lc*: option to save your light curve once you've generated it as a .tsv file.

*load_lc*: if 'y', an attempt is made to load a light curve file you have already generated through this code.

*attributes_only*: when set to 'y', this downloads a planet's attributes without downloading the light curve.

*clobber*: if 'y', any light curve file for the target will be overwritten. Sets *load_lc = 'n'*. If neither *load_lc* nor *clobber* are specified and a light curve for this target already exists, the user will be asked to decide whether the file should be clobbered.


### ATTRIBUTES

**The following attributes are available upon successfully downloading a light curve** (or in some cases after detrending). Hopefully their meanings are mostly self explanatory:

 'Dec',
 'DWstat',
 'NEA_rowidx',
 'NEA_targetname',
 'RA',
 'a_rstar',
 'aliases',
 'depth',
 'duration_days',
 'duration_days_err',
 'duration_hours',
 'duration_hours_err',
 'eccen',
 'eccen_err',
 'errors',
 'flags',
 'fluxes',
 'impact',
 'impact_err',
 'incl',
 'incl_err',
 'insol',
 'insol_err',
 'ldm_a1',
 'ldm_a2',
 'longp',
 'longp_err',
 'mask_multiple',
 'neighbor_dict',
 'neighbor_transit_IDs',
 'neighbor_transit_list',
 'neighbor_transit_times',
 'neighbors',
 'newlc',
 'period',
 'period_err',
 'q1',
 'q2',
 'quarter_transit_dict',
 'quarters',
 'rp_meters',
 'rp_rearth',
 'rp_rearth_err',
 'rp_rjup',
 'rprstar',
 'rprstar_err',
 'rstar_meters',
 'rstar_rsol',
 'savepath',
 'sma_AU',
 'smass',
 'smass_err',
 'target',
 'targetID',
 'target_type',
 'tau0',
 'tau0_err',
 'taus',
 'telescope',
 'times'

*here's a quick explanation for some of the less obvious ones:*
DWstat: the Durbin-Watson statistic (measuring autocorrelation post-detrend)
ldm_a1 and ldm_a2: Limb-darkening coefficients.
neighbor_dict: a dictionary of any other planets in the system. They should be their own light curve objects complete with attributes.
smass: stellar mass
longp: longitude of periapse
mask_multiple: how wide (in units of transit duration) you mask on either side of the transit midtime, for detrending.



**And some functions that can be called on the light curve object...** The format is *lc_object.function(args)*. Many of these are called *under the hood*, but you may desire to call them yourself in some situations. The big ones are *detrend()*, *genLS()*, and *plot_lc()*. Additional explanation of these functions below!

 'correlated_noise_detector()'

 'detrend(self, dmeth='cofiam', save_lc='y', mask_transits='y', mask_neighbors='y', mask_multiple=None, skip_ntqs='n', medfilt_kernel_transit_multiple=5, GP_kernel='ExpSquaredKernel', GP_metric=1.0, max_degree=30, use_holczer='y')'

 'examine_TPF(self, quarters=None, time_lims=None, detrend='y', mask_idxs=None)',

 'find_TTVs(self, show_plot='n', yvar='OCmins', mask_multiple=None)',

 'find_aliases()',

 'find_neighbors(self, is_neighbor='n')',

 'find_planet_row(self, alias=None, row_known='n')',

 'find_taus()',

 'find_transit_quarters(self, locate_neighbor='n')',

 'fit(self, custom_param_dict=None, fitter='multinest', modelcode='LUNA', segment='y', segment_length=500, skip_ntqs='y', model='M', nlive=1000, nwalkers=100, nsteps=10000, resume=True, folded=False)',

 'fold(self, detrended='y', phase_offset=0.0)',

 'genLS(self, show_plot = 'y', compute_fap='n', use_detrend='n', minP=None, maxP=None, LSquarters=None)',

 'gen_batman(self, folded='n')',

 'get_coords()',

 'get_future_transits(self, num_transits=20, output_format='datetime', native_format=None), 

 'get_neighbors(self, save_to_file='y', mask_multiple=None)',

 'get_properties(self, locate_neighbor='n')',

 'initialize_priors(self, modelcode)',

 'mystery_solver(self, tau0, period, duration_hours, neighbor_tau0=None, neighbor_period=None, neighbor_duration_hours=None, neighbor_name='None')',

 'plot_bestmodel(self, fitter, modelcode, folded=False, burnin_pct=0.1)',

 'plot_corner(self, fitter='emcee', modelcode='batman', burnin_pct=0.1)',

 'plot_lc(self, facecolor='LightCoral', edgecolor='k', errorbar='n', quarters='all', folded='n', include_flagged='n', undetrended='y', detrended='y', show_errors='n', show_stats='y',
 show_neighbors='y', mask_multiple=None, show_model='y', show_batman='y', show_model_residuals='y', time_format='native', pltshow='y', phase_offset=0.0, binned='n')',

 'prep_for_CNN(self, save_lc='y', window=6, cnn_len=493, exclude_neighbors='y', flag_neighbors='y', show_plot='n', extra_path_info=None, cnnlc_path=moonpydir+'/cnn_lcs')'.


**additional functions that will work independent of a light curve object can be found in mp_tools.py**. If you sniff around in the source code you may find other useful functions to be utilized in your workflow. For example, *mp_lcfind.py* contains all the functions utilized in downloading the light curves. 


**NOTES:**
The light curve object is designed to be versatile. You can either
a) supply a targetID -- either a KOI, Kepler planet, KIC, K2 planet or EPIC target, or a planet observed by TESS -- and the name of the telescope; or 
b) supply coordinates for an object search

If you choose option (a), you may need to make it explicit somehow which telescope you want to use.
For example, you can either enter a targetID like "Kepler-1625b", "KOI-5084.01", or "KIC4760478", OR
you may enter "1625b", "5084.01", or "4760478" for the targetID and specify the telescope as "kepler".
The code will do its best to determine the telescope. It should also accept "Kepler" as well as "kepler",
and "K2" as well as "k2". TESS targets may be specified by a TOI or a TIC, or in some cases another established name for the target (e.g. WASP-46 or HATS-. If you have already downloaded this light curve, 
you may set load_lc='y' to attempt to load a file you have already generated (handy if you've already 
detrended the light curve and don't want to do it again.)

The coordinate search (b) performs a cone search with a (default) 5 arcsecond radius through Simbad. You may change the 
cone size by adjusting the "search_radius" keyword. Some targets have multiple aliases, and if the first hit
is not either a KOI, kepler planet or KIC star, an attempt will be made to find this name amongst the aliases.
Also note that your options for coord_format are 'degrees' and "sexagesimal", but if you input sexagesimal
without indicating it an attempt is made to recognize this and change the coord_format on the fly. Appropriate
syntax for sexagesimal coordinates is '9h36m43.5s' for RA and '+39d42m46.83s' for Dec. I think you can go arbitrarily
precise with the decimal places but I haven't tested this extensively. Spaces between hours/minutes/seconds and degrees/minutes/seconds
should be OK.

You may also download only select quarters if you wish by supplying an array of quarter numbers in the 'quarters' keyword.
Currently supported formats for the light curve download is "sap" and "pdc". If you wish to download both
(possibly included in a future release) you should just initialize two different light curve objects.

*New June 14 2019* -- Support for downloading TESS light curves is here! You can enter a TOI number, a TIC number, the standard name of a confirmed planet if it was observed by TESS (for example, WASP-46, HATS-3). An attempt is made to identify the candidate if you supply an established alias of the target (for example, a 2MASS ID).  There is also a new method for your lc_object. *lc_object.aliases* will show you all the target aliases listed in SIMBAD.



## PLOT THE DATA.

Plotting the data is simple, and I expect the keywords are all self-explanatory. 

Once you have generated your light curve object (step 1 above), you can plot the light curve simply by calling 

*>>> lc_object.plot_lc(facecolor='LightCoral', edgecolor='k', errorbar='n', quarters='all', folded='n', include_flagged='n', detrended='y', show_errors='n', show_neighbors='n')*

If the light curve has already been detrended, you will see the detrended light curve. IF NOT, you will get a 
warning that the light curve has not yet been detrended and you will see instead the raw light curve. 

*New June 3rd*: if *show_neighbors='y'*, additional transiting planets in the system will be identified, each with their own color and marked with an 'x'.


## GENERATE A LOMB-SCARGLE PERIODOGRAM.

Using Astropy's Lomb-Scargle function you can easily generate a Lomb-Scargle periodogram for every quarter simply by calling

*>>>lc_object.genLS(show_plot='y')*

This method will also generate three new attributes for lc_object: LSperiods, LSpowers, and LSfaps.



## DETREND THE DATA.

The current *working* detrending options are CofiAM (Cosine Filtering Autocorrelation Minimization), a median filter, PolyAM (Polynomial autocorrelation minimization performed on the entire quarter), PolyLOC (polynomial fitting that minimizes a BIC and uses on the times around the transit event), and a simple median filter. Users may also opt for Method Marginalization, which will attempt to use all of the above detrending techniques and marginalize over their differences, producing a light curve that (at least in theory) robust against peculiarities of one detrending method. 

The usage is simple:

*>>> lc_object.detrend(dmeth='cofiam', save_lc='y', mask_transits='y', mask_neighbors='y', skip_ntqs='n', kernel=None, max_degree=30)*


### Keywords

*dmeth*: currently supported are "cofiam", "medfilt", 'polyAM', 'polyLOC', 'methmarg', 'george', and 'untrendy' (though beware the last two are not really working).

*save_lc*: default is on. Note that this will overwrite the light curve file you've generated by default in initializing the object, so that now there are five columns: times, fluxes, errors, and fluxes and errors from the detrending.

*mask_transits*: by default, transits are masked by calculating the transit times (assuming linear ephemeris), and one full transit duration
is masked on either side of the transit midtime. That is, the total mask is twice the width of the transit. 

*mask_neighbors*: by default, transits of other planets in the system will also be masked for the detrending.

*skip_ntqs*: this option allows you to only detrend the quarters that actually contain a transit of the planet you're interested in.
This can be useful with cofiam, for example, since each quarter can take ~1 minute to detrend. Off by default.

*medfilt_kernel_transit_multiple* specifies the how many times the transit duration you want to use as your median filter kernel (default: 5).

*GP_kernel* is the keyword used by the ```george``` package for specifying the Gaussian Process kernel. Default is ExpSquaredKernel. 

*GP_metric* is the metric keyword used in ```george``` for the GP kernel. Default: 1.0

*kernel*: this allows you specify the size of the kernel for median filtering.

*max_degree*: for cofiam, this is maximum order k you'll allow cofiam to explore. Practically speaking anything much above this
because too computationally expensive.




## GET PROPERTIES.
This function queries the NASA Exoplanet Archive
to retrieve your target's impact parameter, transit duration, orbital period, all transit midtimes within the 
dataset baseline, the ratio of radii, and the isolated radii for the planet and the star. Uncertainties
for many of these parameters are also available as a tuple, quoting lower and upper sigmas.

Note that this function should be called automatically when you initialize a light curve object (whether you're
downloading it fresh or retrieving a light curve you already loaded and saved), so these attributes ought to be 
available to you automatically without needing to call the *get_properties()* function.

*New as of May 31st 2019* - *get_properties()* calls the *find_neighbors()* function, which will tell you whether
other transiting planets are known in the system. This is useful if you want to make sure that a potential 
moon signal isn't simply another transiting planet. For now you can call lc_object.neighbors to pull up a list
of other planets in the system. More functionality to come!

The following attributes are supported;
```
lc_object.period # days
lc_object.period_err # tuple
lc_object.tau0 # BKJD
lc_object.tau0_err # tuple
lc_object.impact 
lc_object.impact_err # tuple
lc_object.duration_hours
lc_object.duration_hours_err # tuple 
lc_object.duration_days
lc_object.duration_days_err # tuple
lc_object.rprstar 
lc_object.rprstar_err # tuple
lc_object.rp_rearth # units of Earth radii (native unit on NASA Exoplanet Archive)
lc_object.rp_rearth_err # tuple
lc_object.rp_rjup # units of Jupiter radii (converted without uncertainties)
lc_object.rstar_rsol # units of Solar radii (converted from Rp/Rstar, without propagating uncertainties)
lc_object.depth 
lc_object.taus # all BKJD transit midtimes in the baseline, assuming linear ephemeris
lc_object.neighbors ### identifies other transiting planets in the system, if any. More functionality to come!
```
If an attribute doesn't come with an uncertainty tuple, it's probably because this is a non-native value
and I haven't bothered to propagate the uncertainties. Will try to implement this in the future.

More attributes may be added in the future. The first time you run this function it will download an ascii table
from NASA Exoplanet Archive, but should not download the table again until 24 hours have elapsed.




## FIT A TRANSIT MODEL TO THE LIGHT CURVE.
(NOTE: the LUNA code developed by D. Kipping is not currently available on GitHub. The pyluna.py script is simply
a wrapper for this code).

You may fit a LUNA or BATMAN model to your detrended data using the following command:

*>>> lc_object.fit(custom_param_dict=None, fitter='multinest', modelcode='LUNA', skip_ntqs='y', model='M', nlive=1000, nwalkers=100, nsteps=10000, resume=True, folded=False)*


### Keywords
*custom_param_dict*: you may use this to modify the default parameter dictionary. The form must be param_dict['parameter'] = ['prior_type', (lower_bound, upper_bound)]. 

*fitter*: may be "multinest" or "emcee".

*modelcode*: May be "LUNA" (internal use only right now) or "batman".

*skip_ntqs*: If 'y', your fit will not utilize quarters for which the planet does not transit.

*model*: May be "P", "T", "Z", or "M". Default is "M". See notes below.

*nlive*: number of live points utilized by multinest.

*nwalkers*: Number of walkers utilized by emcee.

*nsteps*: maximum number of steps used in the emcee fit.

*resume*: If True, emcee will attempt to read in the last positions of the walkers and continue from there. If False, *the old mcmc walker record is clobbered!*

*folded*: allows you to fit to a phase-folded light curve. Generally not a good idea as it assumes a period and tau0.

**Notes**
As used in Teachey & Kipping (2018), the four models are as follows:
* (P): a planet-only model that assumes strict linear ephemeris;
* (T): a planet-only model that allows the transit times to be fit individually (maximum 6 transits);
* (Z): a moon model that sets the moon radius to zero (useful for testing the dynamical effects of the moon); and
* (M): a fully physical moon model.

they keyword must be one of the keywords accepted by pyluna or batman: 
[RpRstar, rhostar, bplan, Pplan, tau0, q1, q2, rhoplan, sat_sma, sat_phase, sat_inc, sat_omega, MsatMp, RsatRp, Rstar, long_peri, ecc]

Additional taus (up to 6 total) can be used, but this is only meaningful for fitting individual transit timings through the "T" model. These parameter keys should be labeled 'tau1', 'tau2', 'tau3', etc.

the 'prior_type' may be 'uniform', 'loguniform', 'normal', 'lognormal', 'beta', or 'fixed'. If 'fixed', you must supply a single number (not a tuple) that will be the fixed value for this parameter in all of your runs.

If you wish to keep the default parameters there is no need to supply these.



## PLOT YOUR BEST MODEL.

Once you've run a model fit (either with PyMultiNest or emcee, and with either LUNA or batman) you may wish to overplot your best fit onto the data. This is still under development, but you can use


*lc_object.plot_bestmodel(fitter, modelcode, folded=False, burnin_pct=0.1)*

to generate a plot of the resulting model. Note that the "best fit" for each parameter is simply the median value from the chains.
If you want to do something more sophistcated you'll probably want to go in and manually alter the code. 

As of May 29, 2019 this is only working for runs made with emcee. Pymultinest support is coming soon.


## MAKE A CORNER PLOT.

You can also generate a corner plot based on your emcee chains (multinest support coming soon.) Usage here is:


*lc_object.plot_corner(self, fitter='multinest', modelcode='batman', burnin_pct=0.1)*


This will save a corner plot in the chains directory where your planet chains were saved.



## IDENTIFY TARGET NEIGHBORS

*New June 3, 2019*: Using the *get_neighbors(clobber_lc='y', save_to_file='y')* method, you can automatically grab information supplied by the *get_properties()* method about every other known transiting planet in the target system. These will be contained within the newly created *neighbor_dict* attribute. For example, Suppose we're interested in Kepler-90g. After initializing this object

*>>> k90g = MoonpyLC(targetID='Kepler-90g')*

you may then call

*>>> k90g.get_neighbors()*

At which point you can see this planet has several neighbors:

*>>> k90g.neighbor_dict.keys()*

Which will return an array of keys: *dict_keys(['k90f', 'k90b', 'k90d', 'k90c', 'k90e'])*. 

Each of these keys will then access a separate light curve object stored in the dictionary, which has the same attributes as suppled by *get_properties()*. For example, if you wish to know all the transit times of the neighbor Kepler-90c, you would call

*>>> k90g.neighbor_dict['k90c'].taus*

Two additional columns will be added to your target light curve file: "in_transit" and "transiter". "in_transit" will be either 'y' or 'n', indicating whether any planet (the target OR a neighbor) is expected to be transiting at this time step (based on linear ephemerides). the 'transiter' will be the name of the transiting planet. If there is more than one planet transiting at a time, all planets in transit should be indicated.

**Note:** As of June 3rd, *get_neighbors()* downloads the light curves as it would for any other MoonpyLC object (though if it already exists it will not redownload it). This will be fixed in time to speed up the process. By default, these light curves will be clobbered upon extraction of the relevant data (they are duplicates of your target light curve, so in general the user will not want to keep these). All that remains are the attributes, including the times, fluxes, and errors. 

Also note that *get_neighbors()* is automatically called by the *detrend()* method, and by default the neighbor transits are masked prior to detrending. You may turn this off, but if *mask_neighbors* is on, *mask_transits* will also be set to on, overriding a command to turn it off.


## IDENTIFY TRANSIT TIMING VARIATIONS

*New June 4, 2019*: The method *find_TTVs(show_plot='n', yvar='OCmins', window=2)* may be called to identify **possible** Transit Timing Variations (TTVs) in the system. 

This is a very fast and dirty operation intended to alert the user to the potential presence of TTVs. It works by calculating an approximate transit midtime as a weighted average, using times a few transit durations (defined by *window*) away from the midtime calculated from linear ephemeris. If this is worrying you already, you should probably do your own more rigorous TTV fitting! Or consult this work and catalogue: https://arxiv.org/abs/1606.01744 

This function will generate several new attributes: *lc_object.OCs_min, lc_object.OCs_day, lc_object.OCs_over_dur, lc_object.OC_sig_min, lc_object.OC_sig_day*, and *lc_object.OC_sig_over_dur.*

Perhaps the most useful metric is *OC_sig_over_dur*. This provides the standard deviation of O-C values divided by the transit duration. If this number is large, the transit timings are likely swinging with an amplitude much larger than the duration of the transit itself, and linear ephemeris is a poor approximation. It is up to the user to decide whether these TTVs are 'significant.'

If *show_plots* is set to 'y', a plot of O-C values will be generated vs epoch number. Acceptable values for *yvar* are "OCmins", "OCdays", and "OCdurs".


## Prepare CNN-ready files

*New June 5, 2019*: Teachey et al 2019b (in prep) is utilizing Convolutional Neural Networks (CNNs) to identify potential moon transits in the *Kepler* data. To that end, the new *prep_for_CNN(save_lc='y', window=6, cnn_len=493, exclude_neighbors='y', flag_neighbors='y', show_plot='n')* function prepares your light curves to be fed into a CNN. The function also returns the filepath of the generated light curve array.


### Keywords

* *save_lc*: saves a light curve segment as a numpy array. The array will have either 5 or 6 rows. The first row is the times, second row is raw fluxes, third row is raw errors, fourth row is detrended fluxes, fifth row is detrend errors, and sixth row (if applicable) is an array of flags indicating whether another planet in the system is expected to be transiting at this time step (based on linear ephemeris). if zero, there is not an expected planet transit at this index, if 1, a neighbor transit is expected.

* *window*: in days, the window of time on either side of the target transit midtime that you want to grab for this segment.

* *cnn_len*: Typically CNN inputs must be of uniform size. Therefore, after selecting a window size you will need to further pare down your light curve to some standard length of data points, dictated by the *cnn_len* keyword.

* *exclude_neighbors*: If a neighbor is detected in your time window, this light curve will not be generated and saved.

* *flag_neighbors*: This keyword indicates whether you want the fourth row of neighbor transit flags on your light curve file. Note that *exclude_neighbors* takes priority here, so if you opt to exclude neighbors you will not generate a light curve segment file and therefore not have an array of flags. The user will likely want to set *exclude_neighbors* to 'n' if the presence of neighboring transiting planets is not a deal breaker.

* *show_plot*: If activated, every viable transit segment will be plotted.



