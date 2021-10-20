# Review: Scalable Bayesian phylogenetics

### Content

- `online_inference` contains BEAST XML to recreate analyses from section 2, "Reducing burn-in via 'online' inference"

- `hmc`, short for "Hamiltonian Monte Carlo", contains BEAST XML to recreate analyses from section 3, "Constructing new chains via better MCMC proposals"


### Setting up [BEAST](https://beast.community/) with [BEAGLE](https://beast.community/beagle)

#### Setting up BEAGLE

Please follow the [BEAGLE installation instructions](https://github.com/beagle-dev/beagle-lib).
But get the `hmc-clock` branch.

For Mac users, the following commands will compile the CPU version of BEAGLE.
Follow the [instructions](https://github.com/beagle-dev/beagle-lib) if you need to install any other dependent software.

```
xcode-select --install
brew install libtool autoconf automake
git clone -b hmc-clock https://github.com/beagle-dev/beagle-lib.git
cd beagle-lib
mkdir build
cd build
cmake -DBUILD_CUDA=OFF -DBUILD_OPENCL=OFF ..
sudo make install
```


For Linux users, the commands are similar.

```
sudo apt-get install build-essential autoconf automake libtool git pkg-config openjdk-9-jdk
git clone -b hmc-clock https://github.com/beagle-dev/beagle-lib.git
cd beagle-lib
mkdir build
cd build
cmake -DBUILD_CUDA=OFF -DBUILD_OPENCL=OFF ..
sudo make install
```


The libraries are installed into `/usr/local/lib`.
You can find them by `ls /usr/local/lib/*beagle*`.


#### Setting up BEAST

The following commands will compile the `hmc-clock` branch of BEAST.

```
git clone -b hmc-clock https://github.com/beast-dev/beast-mcmc.git
cd beast-mcmc
ant
```

For Mac users, you may need to install ant by `brew install ant` through [Homebrew](https://brew.sh/).

For Linux users, you can install ant by `sudo apt-get install ant`.

This will compile the `jar` files under `beast-mcmc/build/dist/` where you can find `beast.jar`, `beauti.jar` and `trace.jar`.

### Reproducing the analyses

You may use the following commands to recreate examples from the paper.

Change your working directory to where you want to store the resulting log files first.

```
cd where_you_want_to_save_results
```

#### Online inference example

 * Naive starting point: 720-tip run
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -overwrite where_this_repository_is_stored/online_inference/SC2_720_baseline.xml
	```

* Generate save state: 588 tip run
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -save_every 50000000 -save_stem checkpoint588 -overwrite where_this_repository_is_stored/online_inference/SC2_588_baseline.xml
	```

* Resume from save state: 720-tip run
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -load_state where_this_repository_is_stored/online_inference/start720from588.state -overwrite where_this_repository_is_stored/online_inference/SC2_720_resume.xml
	```

#### HMC to learn branch-specific phenotypic evolution: HIV example

 * uMH (baseline comparison without HMC)
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -overwrite where_this_repository_is_stored/hmc/HIV/hiv_rrw_noHmc.xml
	```

* HMC
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -overwrite where_this_repository_is_stored/hmc/HIV/hiv_hmc.xml
	```

* strict Brownian diffusion example
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -overwrite where_this_repository_is_stored/hmc/HIV/hiv_strictBD.xml
	```

#### HMC to learn divergence time: Sars-CoV-2 example

    * uMH (baseline comparison without HMC)
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -beagle_CPU -load_state where_this_repository_is_stored/hmc/SARS-CoV-2/checkpoint_covid1000_baseline -overwrite where_this_repository_is_stored/hmc/covid_1000_baseline_to_hmc.xml
	```

    * HMC
	```
	java -jar -Djava.library.path=/usr/local/lib where_beast_is_git_cloned/beast-mcmc/build/dist/beast.jar -beagle_CPU -load_state where_this_repository_is_stored/hmc/SARS-CoV-2/checkpoint_covid1000_hmc -overwrite where_this_repository_is_stored/xmls/WNV/covid_1000_hmc.xml
	```

Note where run times are recorded, there are separate XML stored in a file entitled `timing` to report runtime without logging Markov chain states to file. We take this precaution to reduce any discrepancy between examples that could be influenced by variable log frequency.