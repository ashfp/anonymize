# anonymize
This is a general purpose data anonomiser. Whilst there are loads of other open source projects which offer the ability to generate anonomized data; to identify data which might need to be anonomised; or to use ML to identify Personally Identifiable Information (PII) - none seemed to meet the requirements I've had on so many occasions. Basically when doing testing of either integration code or testing a data migration I find the need to perform anonomization which meets the follwing criteria
* Maintain the values in certain fields unchanged (e.g. non-PII fields)
* Replace the values of some fields with 'realistic garbage'. Meaning this sould be similar to the origional data. In this case it's important to provide data which includes a realistic set of  
  * sizes: if the real data contains values ranging from 3 to 300 characters long, always generting something 10 characters long isn't realistic (code which couldn't deal with long strings would not be tested), but neither would setting all fields to the longest value (this would make the data set unrealistically large).
  * range of values: For instance does the real data include only A-Z or other latin alphabet characters (such as appostrphies in surnames like O'Flynn), or non-latin alphabet characters (such as Arabic or Chinese characters) 
  * the pattern which fields follow - in some cases this may not be important but in others it may be important to generate values which meet a certain format (e.g. UK phone numbers must begin with a + or a 0, however no other character other than the first should be a +)
* Replace values with a Token (**consistent** realistic garbage) - random garbage is ok in some cases, but when anonymizing lots of files it may be important to perform anonymisation consistently across multiple records either in the same file or in different files. For instance an invoice number needs to be anonomised not just on the invoice record but also on each invoice line. Generating different anonymous data for each record would break the relationship
* Avoid reversability - ensure that a field cannot be reverse engineered to find the origional, or brute forced (so no hashing on values)
* Allow deanonymization - it may be necessary for those who have access to the anonymized data to raise a query about a particular record with whoever did the anonymization. Therefore such the anonymiser should be able to map an anonymized field back to the origional for troubleshooting purposes.

To meet these requirements this project includes two tools:
1. Anonymizer: Processes one or many source files and based on a set of rule (in a config file) anonymises each field to produce one or many output files
2. Analyser: Analyses a source file and produces a rules file to define the structure and what is 'realistic' (e.g. what range of sizes or what patterns do each field follow). This rules file can be further  modified manually before running the anonymizer - alternatively rules files can be written manually and the analyser avoided completely.

# Getting started
## Running the anonymiser
The anonymiser is run from the command line with the following parameters
- -type: The name of the type of file (e.g. customers)
- -rules: (optional) The name of the rules file (defaults to the name of the type with a .cf extension e.g. customers.cf)
- -output: (optional) The folder to put the resulting files (defaults to ./output/)
- files: a list of files or wildcard to denote the files to be anonymised - output files have the same name but in the output folder

## Running the analyser
The analyser is similarly run from the command line with the following parameters
- -type The name of the type of file (e.g. customers)
- -delimeter (optional but either delimeter/widths is required) specifies the delimeter used to seperate each field in a delimeter seperated file (e.g. a comma in a CSV)
- -widths (optional but either delimeter/widths is required) a comma seperated list of the widths of each field in a fixed width file (e.g. 1,5,1,2 if the file stucture looks like A00000B11)
- -rules (optional) specifies the name of the rules file to create (defaults to the name of the type with a .cf extension e.g. customers.cf)
- files a list of files or a wildcard to denote the files to be analysed

# Configuration
Each rules file contains a header with the following:
* Type - the type of file as specified by the -type parameter
* Delimiter (optional - if not present this denotes fixed width)

The rules configuration file contains a field list with the following values for each field
* Field name: The unique name for each field. If the rules file was generated by the analyser this will be in the form TYPE_field_COUNT with count starting a 1 (e.g. customers_field_1). It's important to rename this if tokenised fields need to be used consistently across file types (e.g. if the accounts and customers files both need a customer ID field then the fieldname in both configuration files must share the same field name)
* Mode: The method of anonomisation applied to this field. This can be:
  * *random* - generate a random value meeting the rules specified below
  * *token* - replace with a token which meets the rules specified below (the first time this value is found a random token will be geneated, but subsequent values will recieve the same token)
  * *origional* - keep the origional value unchanged  
* Width (only for fixed width files): The size of this field in characters
* RegEx: The regular expression which must be satisfied when generating a field
* PresentRatio: The probability of this field being non-empty (all spaces for fixed length or zero length for delimted files) - e.g. 0 means never present, 1 means always contains data, 0.5 means contains data half the time
* LengthRatio: An array of the different lenghts with probabilities for each length (when a field is present). This must total 1 e.g. 10:0.5, 11:0.25, 12:0.25 gives a 50% chance of generating a 10 character long strong and a 25% chance of generating either an 11 or 12 character long string. This is combined with a PresentRatio. So if the present ratio was 0.1, 90% would be blank, 5% would have 10 characters, 2.5% with either 11 or 12 characters.

# Tokenisation
Token values are stored in files in the ./mapping folder. There is one file per tokenised field. The file contains:
* Field name (as defined in the Configuration file)
* Origional value
* Anonomized value

**Note: These files contain _origional_ values and are stored unencrypted so these MUST be handled with the same care as the source files and deleted when no longer needed.**