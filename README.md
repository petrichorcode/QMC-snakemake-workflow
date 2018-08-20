# Snakemake workflow: quantum-monte-carlo
This workflow performs a quantum monte carlo (QMC) calculation using "trial" wave function (WFN)
generated by QCHEM and ORCA.

## Authors

* Vladimir Konjkov

## Usage

### Step 1: Install workflow and additional utilities

Download workflow from [repository](https://github.com/Konjkov/snakerules)

Download [molden2qmc.py](https://github.com/Konjkov/molden2qmc) utility required to convert "trial" WFN from MOLDEN output format to CASINO input format.

Download [multideterminant.py](https://github.com/Konjkov/molden2qmc) utility required to convert multi determinant information available in CASSCF and post-HF methods to CASINO format.

Copy them to the directory in the current path.

Choose program to generate "trial" WFN (QCHEM > 4.0 or ORCA 4.0.1) and install it.


### Step 2: Configure workflow

Configure the workflow according to your needs via editing the file `ORCA/Snakefile` or `QCHEM/Snakefile` depending on what program you will use.

The basic information necessary for calculations is contained in global variables:

* MOLECULES is a list of molecular geometry file names in xyz-format (without extension) located in the `chem_database` directory for which the calculation will be performed.

* METHODS is a list of quantum chemistry methods to obtain a "trial" WFN.

* BASES is a list of bases in which the "trial" function will be represented.

* JASTROWS is a filename of template (without extension) defining JASTROW factor

Depending on the program used to generate "trial" WFN, the following rules are available:

* __ORCA__
    ```
    rule ALL_ORCA:
        input: '{method}/{basis}/{molecule}/gwfn.data'
    ```
        perform ORCA calculaction and generate gwfm.data file and correlation.data in case of CASSCF method.  
        _method_ - method available in ORCA to calculate "trial" WFN like HF, any DFT methods (i.e. B3LYP, CAM-BLYP, PBE0), OO-RI-MP2, CASSCF(N,M) for multideterminant extension.  
        _basis_ - any bases available in ORCA (i.e. cc-pVDZ, aug-cc-pVQZ, def2-SVP).  
        _molecule_ - molecular geometry file names in xyz-format (without extension) located in the `chem_database` directory.  

* __QCHEM__
    * __ALL_QCHEM__  
        perform QCHEM calculaction and generate gwfm.data file and correlation.data in case of OD, OD(2), VOD, VOD(2), QCCD, QCCD(2), VQCCD.  
        General form of this rule is '{method}/{basis}/{molecule}/gwfn.data' where  
        __method__ - method available in QCHEM to calculate "trial" WFN including HF, any DFT methods (i.e. B3LYP, CAM-BLYP, PBE0), OO-RI-MP2, any orbital optimized methods from the list (OD, OD(2), VOD, VOD(2), QCCD, QCCD(2), VQCCD).  
        in case of OO-method T2-amplitudes where used as determinant's weights but some type of active space truncation should be specified:  
        it should be done with two ways:  
          __multideterminant method___X if X > 1 then first X active orbitals were taken.  
          __multideterminant method___X if X < 1 then all orbitals with T2-amplitudes greate then X were taken.  
        __basis__ - any basis available in QCHEM (i.e. cc-pVDZ, aug-cc-pVQZ, def2-SVP).  
        __molecule__ - molecular geometry file names in xyz-format (without extension) located in the `chem_database` directory.  

* __QMC__
    * __ALL_VMC__  
        perform pure VMC calculation (without JASTROW), this rule intended to check whether the conversion of the "trial" WFN to the CASINO format is correct and HF energy is equal to pure VMC one.  
        General form of this rule is '{method}/{basis}/{molecule}/VMC/{nstep}/out' where  
        __nstep__ - number of VMC statistic accumulation steps.  
        This step is not required to calculate the DMC energy.  
    * __ALL_VMC_OPT__  
        perform JASTROW coefficients optimization using some optimization plan.  
        General form of this rule is '{method}/{basis}/{molecule}/VMC_OPT/{opt_plan}/{jastrow}/out' where  
        __opt_plan__ - name (without extension) of CASINO input file in `opt_plan` directory which define JASTROW optimization plan.  
        __jastrow__ - name (without extension) of template for `parameters.casl` file in `casl` directory.  
    * __ALL_VMC_OPT_ENERGY__  
        perform VMC energy calculation with optimized JASTROW coefficients. It may be required if you need the VMC energy with high accuracy.  
        General form of this rule is '{method}/{basis}/{molecule}/VMC_OPT_ENERGY/{opt_plan}/{jastrow}/{nstep}/out' where  
        __nstep__ - number of VMC statistic accumulation steps.  
        This step is not required to calculate the DMC energy.  
    * __ALL_VMC_DMC__  
        perform FN-DMC energy calculation to achive desired accuracy (specified in the variable STD_ERR).  
        General form of this rule is '{method}/{basis}/{molecule}/VMC_DMC/{opt_plan}/{jastrow}/tmax_{dt}_{nconfig}_{i}/out' where  
        __dt__ - part of denominator for calculating the DMC time step using the formula 1.0/(max_Z**2 * 3.0 * __dt__), integer.  
        __nconfig__ - number of configuration in DMC colculation (1024 is recomended)  
        __i__ - stage of DMC calculation (1 - only DMC equilibration and fixed step (50000) DMC accumulation run, 2 - additional DMC accumulation run to achive desired accuracy)  
    * __ALL_VMC_OPT_BF__  
        performs the same actions as __ALL_VMC_OPT__ but with BACKFLOW transformed WFN.  
        It should be noted that the BACKFLOW transformation in CASINO is implemented only up to f-orbitals.  
        General form of this rule is '{path}/VMC_OPT_BF/{opt_plan}/{jastrow}__{backflow}/out'  
        __backflow__ -
    * __ALL_VMC_OPT_ENERGY_BF__  
        perform the same actions as __ALL_VMC_OPT_ENERGY__ but with BACKFLOW transformation.  
        General form of this rule is '{path}/VMC_OPT_ENERGY_BF/{opt_plan}/{jastrow}__{backflow}/{nstep}/out'  
        __nstep__ - number of VMC statistic accumulation steps.  
        This step is not required to calculate the DMC energy.  
    * __ALL_VMC_DMC_BF__  
        perform the same actions as __ALL_VMC_DMC__ but with BACKFLOW transformed WFN.  
        General form of this rule is '{path}/VMC_DMC_BF/{opt_plan}/{jastrow}__{backflow}/tmax_{dt}_{nconfig}_{i}/out'  

### Step 3: Execute workflow

All you need to execute this workflow is to install Snakemake via the [Conda package manager](http://snakemake.readthedocs.io/en/stable/getting_started/installation.html#installation-via-conda). Software needed by this workflow is automatically deployed into isolated environments by Snakemake.

Test your configuration by performing a dry-run via

    snakemake <rule> -n

Execute the workflow locally via

    snakemake <rule>
