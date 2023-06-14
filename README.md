# MATHUSLA_LLPfiles_HXX
LLP production and decay files for simulation of p p -> h -> XX, X -> bb, gg at the HL-LHC in the MATHUSLA (or other transverse LLP) detector

NOTE: the LLP decay files for H_XX are too large for github and I don't feel like wrestling with LFS. Download them all as a zip at this google drive link:
https://drive.google.com/file/d/15HQ95szei6qOVIhvyK4CblzGNh83FtMH/view?usp=sharing


BENCHMARK MODEL:

pp -> h, h -> XX
at 14 TeV HL-LHC
h = 125 GeV higgs boson
X = LLP with mass mX and lifetime ctauX

Free model parameters: 
- Br(h->XX)
- LLP mass mX
- LLP decay length ctauX
- LLP decay mode. For X being a scalar that couples via the Higgs portal with mX > 10 GeV, decay to bbar dominates by > ~ 90%, so can assume this decay mode as default. For comparison, decay files for X->gg are also provided. (Closer match to e.g. a hadrophilic dark photon Zd -> jj or gluon-portal ALP produced in higgs decays.)


LLP PRODUCTION:
=================
Higgs production was simulated in madgraph for both gluon fusion and vector boson fusion

ggF: simulated at lowest order in HEFT model, for 0 + 1 jet matched, pythia8 for showering. Events were reweighted as a function of pT to match NLO + NNLL dsigma/dpT prediction by HqT 2.0. 
 
VBF: simulated at lowest order in SM model, pythia8 for showering. 

Used cross sections by HWG: 
ggF: 54.6 pb
VBF: 4.27 pb
--> total: 58.87.pb

Both decay channels, after pT reweighting of ggF, were combined into a single set of unweighted events representing total higgs production at the HL-LHC (excluding smaller production contributions like Vh, tth).

LLP 4-vectors were obtained by simulating 2-body decay h->XX. This means that each event contributed two LLP 4-vectors. 

For mX = 10, 15, ..., 55 GeV, the files
HXX_LLP4vectors_mX_10_2perevent_unweighted.csv etc
contain the list of 4-vectors with columns E, pX, pY, pZ, with Z = beam axis. 


LLP DECAY:
=========== 
assuming the LLP is a scalar, decay to either bbar or gg was simulated in madgraph + pythia8. 

Files like bb_15.txt
contain LLP decays, with different LLP decays separated by two blank lines
the first row of each event block is the original LLP 4-vector used in the simulation. This can be ignored (since the simulation used for LLP decays does not match the kinematic distribution of actual LLP production at the HL-LHC.)
Subsequent rows contain lists of LLP decay final states IN THE LLP RESTFRAME, in format 
E, pX, pY, pZ, mass, PID, particle name.

note not all PIDs have a 'particle name' entry, that column is just for convenience.


USAGE FOR MATHUSLA GEANT SIMULATIONS
=======================================

Select some LLP mass mX to study, say mX = 20 GeV. 
You will need the corresponding LLP 4-vector file
HXX_LLP4vectors_mX_20_2perevent_unweighted.csv
and the LLP decay products file, say for bb decay mode, 
bb_20.txt

1. Define rapidity and azimuthal angle ranges ((eta_min, eta_max), (phi_min, phi_max)) that roughly bracket the solid angle acceptance of MATHUSLA. (Overestimate so that no part of MATHUSLA falls outside of that range.)

2. Select a subset of LLP 4-vectors called {MCsample} from HXX_LLP4vectors_mX_20_2perevent_unweighted.csv that you want to use in your simulations. Let the number of 4-vectors in this sample be N_MC. 

These N_MC 4-vectors represent LLP production from total LHC higgs production of 58.87.pb, i.e. they represent
N_LLP = (3000/fb) * (58.87.pb) * 2 * Br(h->XX) = 3.53e8 * Br(h->XX)
produced LLPs. Br(h->XX) is an overall normalizing rate that is unknown, and the point of our simulation is to see how small Br can be while still being observable in MATHUSLA. 

Assign each LLP 4-vector i in our sample {MCsample} a weight
w_i = N_LLP / N_MC. 
This weight corresponds to the number of LLPs that this simulated 4-vector represents. 


3. From {MCsample}, select the subset {MCsample_etacut} that has rapidity in the range (eta_min, eta_max).

Furthermore, due to the symmetry around the beam axis, the phi-angle can be freely chosen for each event. For each surviving LLP 4-vector in {MCsample_etacut}, choose a random phi-angle in the range (phi_min, phi_max) and rotate the 4-vector around the beam axis to have that angle. The chance that a random LLP has a phi angle in the MATHUSLA acceptance is 
f_phi = (phi_max - phi_min)/2pi

Give each rotated LLP 4-vector i in our sample {MCsample_etacut} a weight
w'_i = w_i * f_phi. 
This weight corresponds to the number of LLPs flying through (or close to) MATHUSLA that this simulated 4-vector represents. 


4. Now we have to specify an X decay length ctauX in meters. 

For each LLP 4-vector i in {MCsample_etacut}, find lengths along its trajectory (L1, L2) where it enters and exits the MATHUSLA detector volume that we include in our analysis. The chance of the LLP to decay in the detector is then

P_decay_i  = Exp(-L1/(b*ctau)) - Exp(-L2/(b*ctau))

where b = sqrt(px^2+py^2+pz^2)/mX is the boost of the LLP. 

Next, choose an  LLP decay position along the LLP trajectory between L1 and L2 from an exponential distribution
dPdecay/dl = 1/(b*ctau) Exp[-l/(b*ctau)]. 

Finally, choose an LLP decay event from bb_20.txt. Lorentz-transform the decay daughter products into the lab-frame along the LLP momentum direction with boost b. Place them at decay position (x,y,z), and assign this decay event the weight 
w''_i = w'_i * P_decay_i
which is just the number of LLP decays this decay event represents. 

5. Do whatever you do in your GEANT analysis to decide whether MATHUSLA can reconstruct this LLP decay i. If yes, count this LLP decay as w''_i observed LLP decays. 


6. 
FOR RECONSTRUCTION EFFICIENCY STUDIES: 
do the above, separate LLP decay events into groups (reconstructed) and (not reconstructed). 
efficiency = sum_i(reconstructed) w_i / sum_i(reconstructed + not reconstructed) w_i

This efficiency will have a significant dependence on mX, and will in general depend on LLP decay length ctauX. However, in the long lifetime limit (say ctauX > ~ 1km), LLP decays are approximately uniformly distributed along their trajectories in MATHUSLA, and the lifetime will drop out of the efficiency. This efficiency in the 'long lifetime limit' is therefore a particularly interesting quantity to compute in simulations. 


FOR SIGNAL vs BACKGROUND and SENSITIVITY STUDIES:
For each mX, and for each ctauX, obtain the list of observed LLP decays in our simulation, and at the very end find Br(h->XX) such that
sum_i w''_i = 4 (or whatever the minimum number of observed LLP decays for our analysis is)
Numerically, it therefore makes sense to just set Br(h->XX) = 1 in all the weights in the steps above, and insert it as a prefactor into the weight sums at the end. 






 



