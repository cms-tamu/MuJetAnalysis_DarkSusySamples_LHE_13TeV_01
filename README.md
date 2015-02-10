# MuJet Analysis Samples 13 TeV - Part 1

# Instructions

## 1. Set up new process with Madgraph

### 1.1. Download archive with pre-compiled Madgraph from github

wget https://github.com/cms-tamu/MuJetAnalysis_Samples_13TeV_01/blob/master/MG_ME_V4.5.2_CompiledBackup/MG_ME_V4.5.2_CompiledBackup.tar.gz?raw=true -O MG_ME_V4.5.2_CompiledBackup.tar.gz

### 1.2. Unzip and rename it to MG_ME_V4.5.2

tar -xzf MG_ME_V4.5.2_CompiledBackup.tar.gz

mv MG_ME_V4.5.2_CompiledBackup MG_ME_V4.5.2

### 1.3. Go to Madgraph folder:

cd MG_ME_V4.5.2

### 1.4. Create template for the process

Copy the Template directory to directory with new name, for example pp_to_Higgs_HEFT_Model, in order to keep a clean copy of the Template:

cp -r Template pp_to_Higgs_HEFT_Model

### 1.5. Set up process pp -> Higgs through a top loop

See details in http://madgraph.hep.uiuc.edu/EXAMPLES/Cards/proc_card_4.dat on Madgraph web http://madgraph.hep.uiuc.edu/EXAMPLES/proc_card_examples.html

Edit the file pp_to_Higgs_HEFT_Model/Cards/proc_card.dat:

> pp>h  @1           # First Process
QCD=99             # Max QCD couplings
QED=0              # Max QED couplings
HIG=1              # Max HIGGS EFT coupling: (now max is 1)
end_coup           # End the couplings input

NOTE: Don't forget to specify choice of model. In our test case it is heft:

> # Begin MODEL  # This is TAG. Do not modify this line
heft
# End   MODEL  # This is TAG. Do not modify this line

### 1.6. Setup the specified process

cd pp_to_Higgs_HEFT_Model/bin

./newprocess

NOTE: this will replace the file pp_to_Higgs_HEFT_Model/Cards/param_card.dat by the original param_card.dat for the model. In our case model is "heft", so the original file is MG_ME_V4.5.2/Models/heft/param_card.dat.

### 1.7. Check the specified process

Use your web browser, by looking at index.html in the pp_to_Higgs_HEFT_Model folder:

firefox pp_to_Higgs_HEFT_Model/index.html

### 1.8. Specify the model parameters

The model parameters include masses and widths for particles and coupling constants. They are defined in file param_card.dat in the MG_ME_V4.5.2/pp_to_Higgs_HEFT_Model/Cards folder.

In our case adjust mass of Higgs to 125 GeV:

> 25     1.25000000E+02   # H        mass

## 2. Generate Higgs events with Madgraph

### 2.1. Set generation parameters

Set beam type to protons, beam energy (6.5 TeV in this example), number of events and random seed in the file pp_to_Higgs_HEFT_Model/Cards/run_card.dat:

>  #*********************************************************************
# Number of events and rnd seed                                      *
#*********************************************************************
    800000   = nevents ! Number of unweighted events requested 
    1234     = iseed   ! rnd seed (0=assigned automatically=default))
#*********************************************************************
# Collider type and energy                                           *
#*********************************************************************
       1     = lpp1  ! beam 1 type (0=NO PDF)
       1     = lpp2  ! beam 2 type (0=NO PDF)
    6500     = ebeam1  ! beam 1 energy in GeV
    6500     = ebeam2  ! beam 2 energy in GeV

### 2.2. Generate events with the process already set up

cd pp_to_Higgs_HEFT_Model/bin

./generate_events

This program asks 2 questions:

> Enter 2 for multi-core, 1 for parallel, 0 for serial run
0
Enter run name
ggToHiggs_mH_125_13TeV_madgraph452_events80k

Generated events are stored in file MG_ME_V4.5.2/pp_to_Higgs_HEFT_Model/Events/ggToHiggs_mH_125_13TeV_madgraph452_unweighted_events.lhe.gz

Unzip this LHE file with generated <b>unweighted</b> events, it will be used in next steps:

cd MG_ME_V4.5.2/pp_to_Higgs_HEFT_Model/Events

unzip ggToHiggs_mH_125_13TeV_madgraph452_unweighted_events.lhe.gz

Repeat generation for other masses of Higgs. Suggested run names:

ggToHiggs_mH_090_13TeV_madgraph452_events80k
<br>ggToHiggs_mH_100_13TeV_madgraph452_events80k
<br>ggToHiggs_mH_110_13TeV_madgraph452_events80k
<br>ggToHiggs_mH_125_13TeV_madgraph452_events80k
<br>ggToHiggs_mH_150_13TeV_madgraph452_events80k

## 3. Create custom Dark SUSY model

In our example Higgs decays into two neutralinos n2 that decay into dark neutralino n1 (LSP) and dark photon zd. Dark photon decays into two muons.

### 3.1. Create template for the model

Copy folder MG_ME_V4.5.2/Models/usrmod with custom user model to new folder:

cp -r Models/usrmod Models/usrmod_DarkSusy_mH_125

### 3.2. Define model's particles

Edit Models/usrmod_DarkSusy_mH_125/particles.dat:

>  #MODEL EXTENSION
n1      n1        F        S      MN1   WN1     S    n1   3000001
n2      n2        F        S      MN2   WN2     S    n2   3000002
zd      zd        V        W      MZD   WZD     S    zd   3000022
mu1-    mu1+      F        S      MMU1  WMU1    S    mu1  3000013
# END

NOTE: Muon with new code 3000013 to make it massive

### 3.3. Define model's interactions

Edit `Models/usrmod_DarkSusy_mH_125/interaction.dat`.

Add new vertexes:

        #   USRVertex
        n2   n2   h    GHN22   QED
        n2   n1   zd   GZDN12  QED
        mu1- mu1- zd   GZDL    QED

Remove SM Higgs vertexes to exclude Higgs decays to SM particles:

        # FFS (Yukawa)
        #ta- ta- h GHTAU QED
        #b   b   h GHBOT QED
        #t   t   h GHTOP QED

        #   VVS
        #w- w+ h  GWWH  QED
        #z  z  h  GZZH  QED

### 3.4. Convert model

Run the shell script `./ConversionScript.pl`

### 3.5. Redefine model's couplings

Edit file `Models/usrmod_DarkSusy_mH_125/couplings.f`.

Change couplings from default `1` to some small number `0.001` (narrow width approximation):

        c************************************
        c UserMode couplings
        c************************************
             GHN22(1)=dcmplx(1d-3,Zero)
             GHN22(2)=dcmplx(1d-3,Zero)
             GZDN12(1)=dcmplx(1d-3,Zero)
             GZDN12(2)=dcmplx(1d-3,Zero)
             GZDL(1)=dcmplx(1d-3,Zero)
             GZDL(2)=dcmplx(1d-3,Zero)

### 3.6. Re-define particles' masses and decay widths

Edit file `Models/usrmod_DarkSusy_mH_125/param_card.dat`:

1. Adjust mass of Higgs to 125 GeV, mass of n2 to 10 GeV, mass of n1 to 1 GeV, mass of zd to 400 MeV, mass of mu1 to 105 MeV
2. Set widths of stable particles n1 and mu1 set to 0
3. Set Higgs width to 1 and remove branchings to SM particles

             25     1.25000000E+02   # H        mass
        3000001     1.00000000e+00   # MN1
        3000002     1.00000000e+01   # MN2
        3000022     4.00000000e-01   # MZD
        3000013     1.05000000e-01   # MMU1
        #            PDG       Width
        DECAY   3000001     0.00000000e+00   # WN1
        DECAY   3000002     1.00000000e+00   # WN2
        DECAY   3000022     1.00000000e+00   # WZD
        DECAY   3000013     0.00000000e+00   # WMU1
        DECAY         6     1.51013490E+00   # top width
        DECAY        23     2.44639985E+00   # Z   width
        DECAY        24     2.03535570E+00   # W   width
        DECAY        25     1.00000000e+00   # H   width

### 3.7. Compile model's couplings and run

`make couplings`

`./couplings`

## 4. Define decay table using BRIDGE

### 4.1. Go to the BRIDGE folder:

`cd MG_ME_V4.5.2/BRIDGE`

### 4.2. Run BRIDGE

`./runBRI.exe`

The program asks a few following questions:

        Would you like to run from a MadGraph Model directory? (Y/N): Y
        
        What is the name of the model directory(assuming that it is a subdirectory of Models/): usrmod_DarkSusy_mH_125
        
        Do you wish to generate decay tables for all particles listed above or a subset?(type 1 for all, 2 for subset): 2
        
        Please enter particles you wish to create decay tables for, you must explicitly enter antiparticles if you want BRI to generate their decay tables, otherwise use antigrids.pl: (type 'done' when finished):
        h
        n2
        zd
        done
        
        Please enter a random number seed or write 'time' to use the time: 1234
        
        The default number of Vegas calls is 50000. Would you like to change this? (Y/N) n
        
        The default max. number of Vegas iterations is 5. Would you like to change this? (Y/N) n
        
        Would you like to calculate three-body widths even for particles with open 2-body channels? (Y/N) n
        
        Do you wish to replace the values of the param_card.dat widths, with the values stored in slha.out?(Y/N) y
        
        Do you wish to keep a copy of the old param_card.dat?(Y/N) y

NOTE: the file `param_card.dat` was updated with new decay widths:

        DECAY        25   4.77997464e-06   # h decays
        #          BR         NDA      ID1       ID2
             1.00000000e+00    2     3000002   3000002   # BR(h -> n2 n2 )
        #
        DECAY   3000002   1.20714630e-04   # n2 decays
        #          BR         NDA      ID1       ID2
             1.00000000e+00    2     3000001   3000022   # BR(n2 -> n1 zd )
        #
        DECAY   3000022   1.02272608e-08   # zd decays
        #          BR         NDA      ID1       ID2
              1.00000000e+00    2     3000013  -3000013   # BR(zd -> mu1- mu1+ )

## 5. Decay events generated in step 2 within this custom model

### 5.1. Run BRIDGE

`./runDGE.exe`

The program asks a few following questions:

        Would you like to run from a MadGraph Model directory? (Y/N) Y
        
        What is the name of the model directory(assuming that it is a subdirectory of Models/):
             usrmod_DarkSusy_mH_125
        
        What is the name of the input event file(include path if directory is different from where DGE is running)?
             [FUL PATH]/MG_ME_V4.5.2/pp_to_Higgs_HEFT_Model/Events/ggToHiggs_mH_125_13TeV_madgraph452_events80k_unweighted_events.lhe
        
        What is the name of the output event file(include path if directory is different from where DGE is running)?
             DarkSUSY_mH_125_mGammaD_0400_13TeV-madgraph452_bridge224_events80k.lhe
        
        Please enter a random number seed or write 'time' to use the time
             12345
        
        Choose a mode:
        1. Decay a particular particle.
        2. Decay down to a set of final-state particles.
        3. Decay using a specified set of decay modes.
        Which mode? 2
        
        Choose an input method:
        1. Read a file listing final-state particles.
        2. Enter the list of final-state particles manually.
        Which mode? 2

NOTE: In this test example we want to decay particles to mu+mu- final states according to branching ratios.

        Enter a final-state particle name, or "END" to finish: mu1+
        Enter a final-state particle name, or "END" to finish: mu1-
        Enter a final-state particle name, or "END" to finish: END
        Would you like to save the list of final-state particles? (Y/N) Y
        What file name do you want to save to? DarkSUSY_mH_125_mGammaD_0400_13TeV-madgraph452_bridge224_ListFinalStateParticles.txt


### 5.2. Finally, we need to change our custom massive muon "mu1" to regular "mu".

Just search for codes `3000013` and `-3000013` in event file `DarkSUSY_mH_125_mGammaD_2000_13TeV-madgraph452_bridge224_events80k.lhe` and replace them with codes `13` and `-13`, correspondingly.
