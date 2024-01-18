# FFconvert
Protocol for converting pdb from MD simulation to AMBER-type file with corresponding FF. 
# INTRODUCTION
After several procedures with Molecular Dynamics using GROMACS and CHARMM forcefield, we wanted to use specific geometry of MD simulation in QMMM calculations using ORCA.
We decided to use amber forcefield for qmmm calculations, therefore, proper conversion of the files was necessary.
# STEP-BY-STEP SOLUTION
# 1. Save structure using VMD into pdb file. In our case we implied several solvent molecules using selection: _water within 5 of resname LIG_
# 2. Using CHARMM-GUI website, upload the structure and follow the procedure to use AMBER FF (ff19SB)
    The order of atoms in AMBER FF differ from CHARMM FF. This is why it is important to initially create CHARMM files using both of the ForceFields!!!
# 3. Remove excess of solvent.
    VMD trick with few issues: you can open pdb file and delete the frame, however, the information of atoms will be kept. So:
# 4. Open AMBER type pdb file via VMD, delete the frame, and download the file from MD simulation. Save the file.
    ! Caution ! The files must have exact number of atoms!
    As you could understand, the insertion of this kind leads to atom disorders, which we will be working in the next steps.
# 5. Try to use ./tleaprc on the structure obtained from the previous step. Hint: you will certainly have multiple errors. Check the log to identify the problematic cases.
# 6. For now, we will have to replace atomtypes accordingly to be able to run tleaprc with no errors. ! Caution ! tleaprc checks the distance and the atomtypes, thus if atoms are linked or stand nearby each other, this might be skipped with no error signal. Please, check the structure.
# 7. Once every atoms are placed in proper order, go to the end of the formatted pdb file and replace OAT atomtype to OXT (refers to terminal oxygen)
# 8. For disulfide bonds add:
# 9. SSBOND 1 CYX A 197 CYX A 219 # REMARK: TYPE_OF_BOND NUMBER_OF_BOND(you might have multiple) ATOMTYPE_BONDED CHAIN RESIDUE_NUMBER ATOMTYPE_BONDED CHAIN RESIDUE_NUMBER
    Also don`t forget to check your residues corresponding to CYX-CYX: you might have CYS/CYM instead, thus replace by CYX.
    If you have protonated residues such as GLU/ASP - replace them by GLH/ASH - tleaprc recognizes atomtypes and add/remove hydrogens if needed.
# 10. Once we prepared the protein, it is time to prepare the ligand. By using antechamber, type command: 
_antechamber -i 4NP.pdb -fi pdb -nc -2 -fo mol2 -c bcc -o NPH.mol2 -at gaff2_
-nc flag - net charge
-c flag - point charge calculation, here we used AC1-BCC
-at flag - atom type, recomendation of AMBER suggests to use gaff2
# 11. TIP3P was used to model water molecules
# 12. Check for missing parameters through command line:
_parmchk2 -i 4NP.mol2 -f mol2 -o 4NP.frcmod_
# 13. Edit ./leaprc script and run it:
_source leaprc.protein.ff19SB_
_source leaprc.gaff2_
_loadAmberParams ./4NP.frcmod_
_4NP = loadmol2 4NP.mol2_
_WAT = loadmol2 TIP3P.mol2_
_mol = loadpdb AMBER_DUSP5_DA.pdb_
_saveAmberParm mol prmtop inpcrd_
_savepdb mol AMBER_DUSP5_DA.xyz_
# 14. Since we have prmtop file, we are able to generate .prms file for ORCA use:
_$ORCA/orca_mm -convff -AMBER prmtop_
# 15. Voila!  
