playground local
========================

Build Plutus dev environment on Ubuntu 20.04 VM

The Project
Project Title
Build Plutus environment for ubutu 20.04 VM

Details
Purpose / Intention
Build Plutus Dev environment using  Oracle
VM VirtualBox
Desired Outcome(s)
Working dev environment for Plutus

Resources

For good performance:
A computer with VM virtualBox software installed
32 GB of ram
Over 100 GB of hard drive space
Image with Ubuntu 20.04 desktop software

Notes

These directions worked as of 5/12/2021. Things will change over time. Please always check the github repositories listed below for the updated directions and any changes.
These instructions do not follow the github directions as listed, but this is the way I was able to get everything to work. One of the most challenging problems during the beginning of class was getting this working, this is only the way I did it. there are other options.

The directions will be split into 2 sections.
The first section shows you how to install Haskell and needed components directly onto the VM.
The second section shows you how to install the nix-shell and Plutus binaries needed to run the plutus playground.
** You can use the nix-shell for your homework as well. If you decide to do that, You can skip installing Haskell. **


Section 1

1) create VM with at least 8 gig ram and 100 gb of storage. Make sure you pick Ubuntu 64

2) start vm

3) select vm image for ubuntu 20.04

4) go through normal process to install ubuntu on your system (VM)

after you get into your ubuntu:

1) update and upgrade software:

```
	sudo apt update
	sudo apt upgrade -y
```

2) install haskell

```
	sudo apt-get install haskell-platform
```

3) install ghcup and all options (ghcup makes it easy to move between versions of haskell, etc)

```
	sudo apt install curl
	
	sudo apt install build-essential curl libffi-dev libffi6 libgmp-dev libgmp10 libncurses-dev libncurses5 libtinfo5 libsodium-dev

	curl --proto '=https' --tlsv1.2 -sSf https://get-ghcup.haskell.org | sh 

```
-- this could take a while

** RESTART YOUR TERMINAL FIRST **

```
	ghc --version
```

-- it should be: 8.10.4

```
	cabal --version
```
-- it should be: 3.4.0.0


4) clone plutus and plutus pioneer directories. you can put these anywhere. i created a seperate directory to put them in:

```
	sudo apt install git

	mkdir cardano

	cd cardano

	git clone https://github.com/input-output-hk/plutus

	git clone https://github.com/input-output-hk/plutus-pioneer-program
```

*** You now have everything you need to edit smart contracts and run them with the command line interface ***

*** you have to run cabal update and cabal build verytime you start on a different section of the homework ***

*** The first time you run cabal build it will take a long time...

*** you also can now run the repl and test haskel - most of all these examples are in the first lesson video ***


Section 2

*** Now we are going to install nix and  ***

1) we need to install the cache

Create the directory /etc/nix/:

```
	sudo mkdir /etc/nix
```
Use a code editor to put the following in a file called nix.conf in the nix directory (/etc/nix/nix.conf):

```
substituters        = https://hydra.iohk.io https://iohk.cachix.org https://cache.nixos.org/

trusted-public-keys = hydra.iohk.io:f/Ea+s+dFdN+3Y/G+FDgSq+a5NEWhJGzdjvKNGv0/EQ= iohk.cachix.org-1:DpRUyj7h7V830dp/i6Nti+NEO2/nhblbov/8MW7Rqoo= cache.nixos.org-1:6NCHdD59X431o0gWypbMrAURkbJ16ZPMQFGspcDShjY=
```

2) install nix

```
	curl -L https://nixos.org/nix/install | sh
```

* after you install nix make sure to install the environment variables. It will tell you how after nix is finished installing. it will look something

* like this   .  /home/(user)/.nix-profile/etc/profile.d/nix.sh


* you also may need to add nix to your Path. I do it by adding  ~/.nix-profile/bin   to my /etc/environment file. then I have to restart THE COMPUTER

* for it to work


3) run the nix command in the Plutus directory:

*** Before running the following nix-build command, set the tag in plutus back to the original tag (3746610e53654a1167aeb4c6294c6096d16b0502) from the first day of class. This solved a problem I was having with the client not compiling correctly with newer dev environment creations ***

```
	~/cardano/plutus $ git checkout 3746610e53654a1167aeb4c6294c6096d16b0502
```

``` 
	~/cardano/plutus $ nix build -f default.nix plutus.haskell.packages.plutus-core.components.library
```

-- this will take a long time and you will get a warning about: dumping very large path.  You can ignore that.


** once this is done everything is installed except the code editor you are going to use...

** the first time you run nix-shell it will also take a while

** the first video walks you through the startup of the plutus playground server and application
