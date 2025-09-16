# Elliptica Initial Data Reader

This repository is a stand-alone library and provides an interface for an evolution code to
read the initial data of *Elliptica*.

Requirements:
- The GNU make tool.
- A C programming compiler, for instance, `gcc`.
- A compatible `openmp` library with the compiler.

Usage:
1. We choose the compiler at the `GNUmakefile` file. For example, a `gcc` compiler is 
chosen by `CC = gcc` and for the `openmp` library we set `CFLAGS += -fopenmp`.

2. We compile this library by issuing the command `make`.

3. To link this library to *an evolution code*, one needs to pass the path `-L/path/to/Elliptica_ID_Reader/lib` 
to the compiler during the compilation of the evolution code. 
Similarly, the compiler requires the path `-I/path/to/Elliptica_ID_Reader/include` 
to find the pertinent header file.


## Tutorial
How to use ID reader in an evolution code with a Cartesian grid.

```c

const char *checkpnt_path  = "full/path/to/elliptica/checkpoint/file"
const int Npnts = 16*16*16; // for all x,y,z coords

// initialize
Elliptica_ID_Reader_T *idr = elliptica_id_reader_init(checkpnt_path,"generic");

// the list of fields to be interpolated. should be comma separated
idr->ifields  = "alpha,betax,betay,betaz,adm_gxx,adm_gxy";
idr->npoints  = Npnts;
// here we convert a 3D index to 1D index, e.g., (i,j,k) -> ijk.
idr->x_coords = a pointer to double type 1D(ijk) array of Cartesian x coord values;
idr->y_coords = a pointer to double type 1D(ijk) array of Cartesian y coord values;
idr->z_coords = a pointer to double type 1D(ijk) array of Cartesian z coord values;

// set parameters for elliptica. These won't change Elliptica's checkpoint file.
// example:
idr->set_param("BH_filler_method","ChebTn_Ylm_perfect_s2",idr);
idr->set_param("ADM_B1I_form","zero",idr);

// get double parameters for elliptica, for example:
idr->get_param_dbl("BHNS_angular_velocity",idr);

// interpolate
elliptica_id_reader_interpolate(idr);

// now in an evolution code one can get interpolated values as follows:
int ijk = 0;
for(i,j,k)
{
    double g_xx  = idr->field[idr->indx("adm_gxx")][ijk];
    double betax = idr->field[idr->indx("betax")][ijk];
    double alpha = idr->field[idr->indx("alpha")][ijk];
    
    ijk++;
}

// free
elliptica_id_reader_free(idr);

// note: for optimization inside the for(i,j,k) loop, 
// one can save the field indices beforehand like:

const int iell_alpha     = idr->indx("alpha");
const int iell_betax     = idr->indx("betax");
const int iell_betay     = idr->indx("betay");
const int iell_betaz     = idr->indx("betaz");

const int iell_adm_gxx   = idr->indx("adm_gxx");
const int iell_adm_gxy   = idr->indx("adm_gxy");
const int iell_adm_gxz   = idr->indx("adm_gxz");
const int iell_adm_gyy   = idr->indx("adm_gyy");
const int iell_adm_gyz   = idr->indx("adm_gyz");
const int iell_adm_gzz   = idr->indx("adm_gzz");

const int iell_adm_Kxx   = idr->indx("adm_Kxx");
const int iell_adm_Kxy   = idr->indx("adm_Kxy");
const int iell_adm_Kxz   = idr->indx("adm_Kxz");
const int iell_adm_Kyy   = idr->indx("adm_Kyy");
const int iell_adm_Kyz   = idr->indx("adm_Kyz");
const int iell_adm_Kzz   = idr->indx("adm_Kzz");

const int iell_grhd_rho  = idr->indx("grhd_rho");
const int iell_grhd_epsl = idr->indx("grhd_epsl");
const int iell_grhd_p    = idr->indx("grhd_p");
const int iell_grhd_vx   = idr->indx("grhd_vx");
const int iell_grhd_vy   = idr->indx("grhd_vy");
const int iell_grhd_vz   = idr->indx("grhd_vz");

//----------------------------------------------------------------------
// if you want a thread safe interpolation
//----------------------------------------------------------------------

// initialize (not thread safe)
Elliptica_ID_Reader_T *idr = elliptica_id_reader_init(checkpnt_path,"generic_MT_safe");

// this is needed only for BHNS system
idr->set_param("BH_filler_method","ChebTn_Ylm_perfect_s2",idr);

// this is needed for both BHNS and BNS systems
idr->set_param("ADM_B1I_form","zero",idr);

// preparation and setting some interpolation settings (not thread safe)
elliptica_id_reader_interpolate(idr);

// now in an evolution code one can get interpolated values as follows:
for(i,j,k)
{
    // set x,y,z coord of the evolution code.
    double x = x_coord_of_evo_code(...);
    double y = y_coord_of_evo_code(...);
    double z = z_coord_of_evo_code(...);
    
    // populate point wise. (THREAD SAFE)
    double g_xx  = idr->fieldx(idr,"adm_gxx",x,y,z);
    double betax = idr->fieldx(idr,"betax",x,y,z);
    double alpha = idr->fieldx(idr,"alpha",x,y,z);
    
    ijk++;
}

// free (thread safe)
elliptica_id_reader_free(idr);


```
