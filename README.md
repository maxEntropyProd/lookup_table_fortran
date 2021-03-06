# Lookup table

A [lookup table](https://en.wikipedia.org/wiki/Lookup_table) is an array that
can replace an expensive computation with a linear interpolation. A good way to
understand the usage of this module is to run:

    make
    ./usage_example

After which you can inspect the file `usage_example.f90` to see how you can
incorporate the module in your own programs.

## Requirements

A modern Fortran compiler, such as `gfortran 4.8` or newer.

## The lookup table type

The package `lookup_table` can easily be used in program environments where the
use of lookup tables makes sense. The tables may be precalculated and stored in
a `lookup_table_t` type defined as

    !> The lookup table type. There can be one or more columns, for which values
    !> can be looked up for a given 'x-coordinate'.
    type lookup_table_t
       integer  :: n_rows !< The number of rows
       integer  :: n_cols !< The number of columns
       real(dp) :: x_min  !< The minimum lookup coordinate
       real(dp) :: dx     !< The x-spacing in the lookup coordinate
       real(dp) :: inv_dx !< The inverse x-spacing

       ! The table is stored in two ways, to speed up different types of lookups.
       real(dp), allocatable :: cols_rows(:, :) !< The table in column-major order
       real(dp), allocatable :: rows_cols(:, :) !< The table in row-major order
    end type lookup_table_t

## Creating a lookup table

Lookup tables are created with a call to the function `LT_create`:

    lt = LT_create(x_min, x_max, n_rows, n_cols)

Here `n_rows` controls how many `x`-values are stored. The spacing of the
(linear) lookup table is therefore given by:

    dx = (x_max - x_min) / (n_rows - 1)

The argument `n_cols` specifies how many different columns you want to store,
but it also possible to add extra columns later with `LT_add_col`.

The columns of a table are set using `LT_set_col`:

    call LT_set_col(lt, col_ix, x, y)

where `x` and `y` are vectors of the same length, with the only requirement that
the `x` array is sorted from low to high. These `y` values will be linearly
interpolated and stored in the table.

## Getting values from a table

There are multiple ways to get values from a lookup table:

* You can directly look up a single column at some `x`-coordinate, using
  `LT_get_col`
* You can get all the columns at some `x`-coordinate, using
  `LT_get_mcol`
* You can also get a location in the table with a `LT_loc_t` object, and then do
  lookups using that object. This can be used to speed up multiple lookups of
  different columns

The type `lt_loc_t` is defined as:

    type LT_loc_t
       private
       integer  :: low_ix   !< The x-value lies between low_ix and low_ix+1
       real(dp) :: low_frac !< The distance from low_ix (up to low_ix+1), given
                            !< as a real number between 0 and 1.
    end type LT_loc_t

The values in the table are calculated once only as part of a program's
initialization phase. To transform the input of the table to regularly spaced
data, we need some subroutines for finding indices in sorted lists. Such
subroutines can be found in module `m_find_index.f90`

    !> Linear search of sorted list for the smallest ix such that list(ix) >= val.
    !> On failure, returns size(list)+1
    function find_index_linear(list, val) result(ix)

    !> Binary search of sorted list for the smallest ix such that list(ix) >= val.
    !> On failure, returns size(list)+1
    function find_index_bsearch(list, val) result(ix)

    !> Adaptive search (combination of linear and binary search) of sorted list
    !> for the smallest ix such that list(ix) >= val. On failure, returns
    !> size(list)+1
    function find_index_adaptive(list, val) result(ix)

## Public methods

The following public functions and subroutines are presented in module
`m_lookup_table`:

    ! Public methods
    public :: LT_create           ! Create a new lookup table
    public :: LT_get_xdata        ! Get the x-values of a table
    public :: LT_get_spaced_data  ! Convert values to regularly spaced
    public :: LT_set_col          ! Set one table column
    public :: LT_add_col          ! Add a column
    public :: LT_get_loc          ! Get the index (row) of a value
    public :: LT_get_col          ! Interpolate one column
    public :: LT_get_mcol         ! Interpolate multiple columns
    public :: LT_get_col_at_loc   ! Get one column at location
    public :: LT_get_mcol_at_loc  ! Get multiple columns at location
    public :: LT_get_data         ! Get all the data of the table
    public :: LT_lin_interp_list  ! Linearly interpolate a list
    public :: LT_to_file          ! Store lookup table in file
    public :: LT_from_file        ! Restore lookup table from file

## Tests

An example how to add data is shown in the test programs

* `test_lookup_table_performance`
* `test_find_index_performance`
* `test_find_index_simple`

## Contributors

* [Jannis Teunissen](http://teunissen.net/): Main author, supported
  by [STW](http://www.stw.nl/) while working at [CWI](https://www.cwi.nl/) and
  now by [FWO](http://www.fwo.be/) while working
  at [KU Leuven](https://wis.kuleuven.be/CmPA).
* [Margreet Nool](https://www.cwi.nl/people/211): Documentation

## TODO

* Add option to use `ifort` compiler in `Makefile`
