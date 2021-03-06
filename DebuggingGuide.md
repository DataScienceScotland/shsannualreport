
<!-- DebuggingGuide.md is generated from DebuggingGuide.Rmd. Please edit that file -->

# shsannualreport - Debugging Guide

*The debugging process described here can be time consuming, so it’s
worth pointing out to start with that errors are often visible in the
raw data, so it makes sense to check the Excel sheets for any obvious
errors before going through this process.*

Each of the exported functions in `shsannualreport` calls a number of
internal functions, with output written to the console to inform the
user of progress. If errors occur it should be possible to see which
function it occured in and debug accordingly.

However, some functions work by looping through a number of excel
sheets, reading the sheets to memory and processing them by creating
strings of R code dynamically. This can be difficult to debug as the
data and executed code are only in memory and so not accessible by the
user.

One example of this is in the function `shs_create_app_data`, which
calls internal functions including `shs_process_dataset`, which then
calls other internal functions including `shs_process_table_type_1`.

When an error occurs in `shs_process_table_type_1`, there is no
immediate way for a user to check either the state of the data at that
point, or the code that has been executed. To debug, it’s neecessary to
manually step through the code that’s been run to that point, and save
the data and the generated code off elsewhere to debug.

## Steps

The steps to do this are:

  - Create an new folder to save test data to.
  - Clone or download this [GitHub
    repo](https://github.com/DataScienceScotland/shsannualreport) and
    open the project in R Studio.
  - Arrange the source data and metadata as described in the
    [readme](https://github.com/DataScienceScotland/shsannualreport/blob/master/README.md),
    and assign the variables `destination_directory`,
    `source_data_directory`, `columns_to_remove`,
    `column_names_workbook_path`, `variable_names_workbook_path`
    accordingly.
  - Open `R/shs_create_app_data.R` and starting from the first
    `tryCatch` command run each command individually up to and including
    the one that contains `shs_process_variable_names`.
  - Open the file `R/shs_process_dataset.R`
  - Run the commands at the start of the function to assign the
    variables `question_titles`, `data_files`, `design_factors_path`,
    `type_1_tables`, `type_2_tables`, `type_3_tables` and
    `type_4_tables`.

*The following instructions are for type 1 tables specifically but can
be modified for other table types*

  - Print the list of `type_1_tables` and select one to debug. Assign
    the value to a new R variable named `table`. (In the code this would
    be assigned by looping through all type 1 tables, assigning one
    desired value allows us to step through the process without
    looping.)
  - Run the command to assign the variable `save_file_path`.
  - Run the if statement that immediately follows (`if (grepl(", ",
    table)) {`… There are a number of conditions that will run here,
    covering about 40 lines of the script.)
  - Open the file `R/shs_process_table_type_1.R`.
  - Run the commands to assign the variables `df` and `design`.
  - Save both `df` and `design` to the folder created for test data
    using `saveRDS`.
  - Run through the subsequent commands in
    `R/shs_process_table_type_1.R`, stopping before the first `eval`
    command.
  - Write the code strings assigned to each variable (`main_df_string`,
    `values_df_string`, `sig_lower_df_string`, `sig_upper_df_string`,
    and `final_df_string`) to a new R script in the test data folder.
    (This can be done programatically or by printing to the console and
    using copy/paste, but must follow the order the variables are listed
    above.)
  - Format the code in the file to remove quotations/escape characters
    etc.
  - In a new session, load in the test .Rds data from the test folder,
    and run the code.

Any errors can be now be debugged. If any changes are then required in
the original function, the debugging script can be used as the basis to
update the code, bearing in mind that as the code is being generated as
a string and then evaluated it me be necessary to escape some
characters, concatenate new lines etc.
