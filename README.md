# Table of Contents
1. [Introduction](README.md#introduction)
2. [Challenge summary](README.md#challenge-summary)
3. [Details of challenge](README.md#details-of-challenge)
4. [Input file](README.md#input-file)
5. [Output files](README.md#output-files)
6. [Example](README.md#example)
7. [Writing clean, scalable and well-tested code](README.md#writing-clean-scalable-and-well-tested-code)
8. [Repo directory structure](README.md#repo-directory-structure)
9. [Testing your directory structure and output format](README.md#testing-your-directory-structure-and-output-format)
10. [Instructions to submit your solution](README.md#instructions-to-submit-your-solution)
11. [FAQ](README.md#faq)

# Introduction
You’re a data engineer working for political consultants and you’ve been asked to help identify possible donors for a variety of upcoming election campaigns. 

The Federal Election Commission regularly publishes campaign contributions and while you don’t want to pull specific donors from those files — because using that information for fundraising or commercial purposes is illegal — you want to identify the areas (zip codes) that may be fertile ground for soliciting future donations for similar candidates. 

Because those donations may come from specific events (e.g., high-dollar fundraising dinners) but aren’t marked as such in the data, you also want to identify which time periods are particularly lucrative so that an analyst might later correlate them to specific fundraising events.

# Challenge summary

For this challenge, we're asking you to take an input file that lists campaign contributions by individual donors and distill it into two output files:

1. `medianvals_by_zip.txt`: contains a calculated running median, total dollar amount and total number of contributions by recipient and zip code

2. `medianvals_by_date.txt`: has the calculated median, total dollar amount and total number of contributions by recipient and date.

As part of the team working on the project, another developer has been placed in charge of building the graphical user interface, which consists of two dashboards. The first would show the zip codes that are particularly generous to a recipient over time while the second would display the days that were lucrative for each recipient. 

Your role on the project is to work on the data pipeline that will hand off the information to the front-end. As the backend data engineer, you do **not** need to display the data or work on the dashboard but you do need to provide the information.

You can assume there is another process that takes what is written to both files and sends it to the front-end. If we were building this pipeline in real life, we’d probably have another mechanism to send the output to the GUI rather than writing to a file. However for the purposes of grading this challenge, we just want you to write the output to files.



# Details of challenge

You’re given one input file, `itcont.txt`. Each line of the input file contains information about a campaign contribution that was made on a particular date from a donor to a political campaign, committee or other similar entity. Out of the many fields listed on the pipe-delimited line, you’re primarily interested in the zip code associated with the donor, amount contributed, date of the transaction and ID of the recipient.

Your code should process each line of the input file as if that record was sequentially streaming into your program. For each input file line, calculate the running median of contributions, total number of transactions and total amount of contributions streaming in so far for that recipient and zip code. The calculated fields should then be formatted into a pipe-delimited line and written to an output file named `medianvals_by_zip.txt` in the same order as the input line appeared in the input file. 

Your program also should write to a second output file named `medianvals_by_date.txt`. Each line of this second output file should list every unique combination of date and recipient from the input file and then the calculated total contributions and median contribution for that combination of date and recipient. 

The fields on each pipe-delimited line of `medianvals_by_date.txt` should be date, recipient, total number of transactions, total amount of contributions and median contribution. Unlike the first output file, this second output file should have lines sorted alphabetical by recipient and then chronologically by date.

Also, unlike the first output file, every line in the `medianvals_by_date.txt` file should be represented by a unique combination of day and recipient -- there should be no duplicates. 


## Input file

The Federal Election Commission provides data files stretching back years and is [regularly updated](http://classic.fec.gov/finance/disclosure/ftpdet.shtml)

For the purposes of this challenge, we’re interested in individual contributions. While you're welcome to run your program using the data files found at the FEC's website, you should not assume that we'll be testing your program on any of those data files or that the lines will be in the same order as what can be found in those files. Our test data files, however, will conform to the data dictionary [as described by the FEC](http://classic.fec.gov/finance/disclosure/metadata/DataDictionaryContributionsbyIndividuals.shtml).

Also, while there are many fields in the file that may be interesting, below are the ones that you’ll need to complete this challenge:

* `CMTE_ID`: identifies the flier, which for our purposes is the recipient of this contribution
* `ZIP_CODE`:  zip code of the contributor (we only want the first five digits/characters)
* `TRANSACTION_DT`: date of the transaction
* `TRANSACTION_AMT`: amount of the transaction
* `OTHER_ID`: a field that denotes whether contribution came from a person or an entity 

### Input file considerations

Here are some considerations to keep in mind:
1. Because we are only interested in individual contributions, we only want records that have the field, `OTHER_ID`, set to empty. If the `OTHER_ID` field contains any other value, ignore the entire record and don't include it in any calculation
2. If `TRANSACTION_DT` is an invalid date (e.g., empty, malformed), you should still take the record into consideration when outputting the results of `medianvals_by_zip.txt` but completely ignore the record when calculating values for `medianvals_by_date.txt`
3. While the data dictionary has the `ZIP_CODE` occupying nine characters, for the purposes of the challenge, we only consider the first five characters of the field as the zip code
4. If `ZIP_CODE` is an invalid zipcode (i.e., empty, fewer than five digits), you should still take the record into consideration when outputting the results of `medianvals_by_date.txt` but completely ignore the record when calculating values for `medianvals_by_zip.txt`
5. If any lines in the input file contains empty cells in the `CMTE_ID` or `TRANSACTION_AMT` fields, you should ignore and skip the record and not take it into consideration when making any calculations for the output files
6. Except for the considerations noted above with respect to `CMTE_ID`, `ZIP_CODE`, `TRANSACTION_DT`, `TRANSACTION_AMT`, `OTHER_ID`, data in any of the other fields (whether the data is valid, malformed, or empty) should not affect your processing. That is, as long as the four previously noted considerations apply, you should process the record as if it was a valid, newly arriving transaction. (For instance, campaigns sometimes retransmit transactions as amendments, however, for the purposes of this challenge, you can ignore that distinction and treat all of the lines as if they were new)
7. For the purposes of this challenge, you can assume the input file follows the data dictionary noted by the FEC for the 2015-current election years
8. The transactions noted in the input file are not in any particular order, and in fact, can be out of order chronologically

## Output files

For the two output files that your program will create, the fields on each line should be separated by a `|`

**`medianvals_by_zip.txt`**

The first output file `medianvals_by_zip.txt` should contain the same number of lines or records as the input data file minus any records that were ignored as a result of the 'Input file considerations.'

Each line of this file should contain these fields:
* recipient of the contribution (or `CMTE_ID` from the input file)
* 5-digit zip code of the contributor (or the first five characters of the `ZIP_CODE` field from the input file)
* running median of contributions received by recipient from the contributor's zip code streamed in so far. Median calculations should be rounded to the whole dollar (drop anything below $.50 and round anything from $.50 and up to the next dollar) 
* total number of transactions received by recipient from the contributor's zip code streamed in so far
* total amount of contributions received by recipient from the contributor's zip code streamed in so far

When creating this output file, you can choose to process the input data file line by line, in small batches or all at once depending on which method you believe to be the best given the challenge description. However, when calculating the running median, total number of transactions and total amount of contributions, you should only take into account the input data that has streamed in so far -- in other words, from the top of the input file to the current line. See the below example for more guidance.

**`medianvals_by_date.txt`**

Each line of this file should contain these fields:
* recipeint of the contribution (or `CMTE_ID` from the input file)
* date of the contribution (or `TRANSACTION_DT` from the input file)
* median of contributions received by recipient on that date. Median calculations should be rounded to the whole dollar (drop anything below $.50 and round anything from $.50 and up to the next dollar) 
* total number of transactions received by recipient on that date
* total amount of contributions received by recipient on that date

This second output file does not depend on the order of the input file, and in fact should be sorted alphabetical by recipient and then chronologically by date.

# Example

Suppose your input file contained only the following few lines. Note that the fields we are interested in are in **bold** below but will not be like that in the input file. There's also an extra new line between records below, but the input file won't have that.

> **C00629618**|N|TER|P|201701230300133512|15C|IND|PEREZ, JOHN A|LOS ANGELES|CA|**90017**|PRINCIPAL|DOUBLE NICKEL ADVISORS|**01032017**|**40**|**H6CA34245**|SA01251735122|1141239|||2012520171368850783

> **C00177436**|N|M2|P|201702039042410894|15|IND|DEEHAN, WILLIAM N|ALPHARETTA|GA|**300047357**|UNUM|SVP, SALES, CL|**01312017**|**384**||PR2283873845050|1147350||P/R DEDUCTION ($192.00 BI-WEEKLY)|4020820171370029337

> **C00384818**|N|M2|P|201702039042412112|15|IND|ABBOTT, JOSEPH|WOONSOCKET|RI|**028956146**|CVS HEALTH|VP, RETAIL PHARMACY OPS|**01122017**|**250**||2017020211435-887|1147467|||4020820171370030285

> **C00177436**|N|M2|P|201702039042410893|15|IND|SABOURIN, JAMES|LOOKOUT MOUNTAIN|GA|**307502818**|UNUM|SVP, CORPORATE COMMUNICATIONS|**01312017**|**230**||PR1890575345050|1147350||P/R DEDUCTION ($115.00 BI-WEEKLY)|4020820171370029335

> **C00177436**|N|M2|P|201702039042410895|15|IND|JEROME, CHRISTOPHER|FALMOUTH|ME|**041051896**|UNUM|EVP, GLOBAL SERVICES|**01312017**|**384**||PR2283905245050|1147350||P/R DEDUCTION ($192.00 BI-WEEKLY)|4020820171370029342

> **C00384818**|N|M2|P|201702039042412112|15|IND|BAKER, SCOTT|WOONSOCKET|RI|**028956146**|CVS HEALTH|EVP, HEAD OF RETAIL OPERATIONS|**01122017**|**333**||2017020211435-910|1147467|||4020820171370030287

> **C00177436**|N|M2|P|201702039042410894|15|IND|FOLEY, JOSEPH|FALMOUTH|ME|**041051935**|UNUM|SVP, CORP MKTG & PUBLIC RELAT.|**01312017**|**384**||PR2283904845050|1147350||P/R DEDUCTION ($192.00 BI-WEEKLY)|4020820171370029339

If we were to pick the relevant fields from each line, here is what we would record for each line.

    1.
    CMTE_ID: C00629618
    ZIP_CODE: 90017
    TRANSACTION_DT: 01032017
    TRANSACTION_AMT: 40
    OTHER_ID: H6CA34245

    2.
    CMTE_ID: C00177436
    ZIP_CODE: 30004
    TRANSACTION_DT: 01312017
    TRANSACTION_AMT: 384
    OTHER_ID: empty

    3. 
    CMTE_ID: C00384818
    ZIP_CODE: 02895
    TRANSACTION_DT: 01122017
    TRANSACTION_AMT: 250
    OTHER_ID: empty

    4.
    CMTE_ID: C00177436
    ZIP_CODE: 30750
    TRANSACTION_DT: 01312017
    TRANSACTION_AMT: 230
    OTHER_ID: empty

    5.
    CMTE_ID: C00177436
    ZIP_CODE: 04105
    TRANSACTION_DT: 01312017
    TRANSACTION_AMT: 384
    OTHER_ID: empty

    6.
    CMTE_ID: C00384818
    ZIP_CODE: 02895
    TRANSACTION_DT: 01122017
    TRANSACTION_AMT: 333
    OTHER_ID: empty

    7.
    CMTE_ID: C00177436
    ZIP_CODE: 04105
    TRANSACTION_DT: 01312017
    TRANSACTION_AMT: 384
    OTHER_ID: empty



We would ignore the first record because the `OTHER_ID` field contains data and is not empty. Moving to the next record, we would write out the first line of `medianvals_by_zip.txt` to be:

`C00177436|30004|384|1|384`

Note that because we have only seen one record streaming in for that recipient and zip code, the running median amount of contribution and total amount of contribution is `384`. 

Looking through the other lines, note that there are only two recipients for all of the records we're interested in our input file (minus the first line that was ignored due to non-null value of `OTHER_ID`). 

Also note that there are two records with the recipient `C00177436` and zip code of `04105` totaling $768 in contributions while the recipient `C00384818` and zip code `02895` has two contributions totaling $583 (250 + 333) and a median of $292 (583/2 = 291.5 or 292 when rounded up) 

Processing all of the input lines, the entire contents of `medianvals_by_zip.txt` would be:

    C00177436|30004|384|1|384
    C00384818|02895|250|1|250
    C00177436|30750|230|1|230
    C00177436|04105|384|1|384
    C00384818|02895|292|2|583
    C00177436|04105|384|2|768

If we drop the zip code, there are four records with the same recipient, `C00177436`, and date of `01312017`. Their total amount of contributions is $1,382. 

For the recipient, `C00384818`, there are two records with the date `01122017` and total contribution of $583 and median of $292.

As a result, `medianvals_by_date.txt` would contain these lines in this order:

    C00177436|01312017|384|4|1382
    C00384818|01122017|292|2|583

## Writing clean, scalable and well-tested code

As a data engineer, it’s important that you write clean, well-documented code that scales for large amounts of data. For this reason, it’s important to ensure that your solution works well for a large number of records, rather than just the above example.

It's also important to use software engineering best practices like unit tests, especially since data is not always clean and predictable. For more details about the implementation, please refer to the FAQ below. If further clarification is necessary, email us at <cc@insightdataengineering.com>

Before submitting your solution you should summarize your approach, dependencies and run instructions (if any) in your `README`.

You may write your solution in any mainstream programming language such as C, C++, C#, Clojure, Erlang, Go, Haskell, Java, Python, Ruby, or Scala. Once completed, submit a link to a Github repo with your source code.

In addition to the source code, the top-most directory of your repo must include the `input` and `output` directories, and a shell script named `run.sh` that compiles and runs the program(s) that implement the required features.

If your solution requires additional libraries, environments, or dependencies, you must specify these in your `README` documentation. See the figure below for the required structure of the top-most directory in your repo, or simply clone this repo.

## Repo directory structure

The directory structure for your repo should look like this:

    ├── README.md 
    ├── run.sh
    ├── src
    │   └── find_political_donors.py
    ├── input
    │   └── itcont.txt
    ├── output
    |   └── medianvals_by_zip.txt
    |   └── medianvals_by_date.txt
    ├── insight_testsuite
        └── run_tests.sh
        └── tests
            └── test_1
            |   ├── input
            |   │   └── itcont.txt
            |   |__ output
            |   │   └── medianvals_by_zip.txt
            |   |__ └── medianvals_by_date.txt
            ├── your-own-test
                ├── input
                │   └── your-own-input.txt
                |── output
                    └── medianvals_by_zip.txt
                    └── medianvals_by_date.txt

**Don't fork this repo*, and don't use this `README` instead of your own. The content of `src` does not need to be a single file called `find_political_donors.py`, which is only an example. Instead, you should include your own source files and give them expressive names.

## Testing your directory structure and output format

To make sure that your code has the correct directory structure and the format of the output files are correct, we have included a test script called `run_tests.sh` in the `insight_testsuite` folder.

The tests are stored simply as text files under the `insight_testsuite/tests` folder. Each test should have a separate folder with an `input` folder for `itcont.txt` and an `output` folder for output corresponding to that test.

You can run the test with the following command from within the `insight_testsuite` folder:

    insight_testsuite~$ ./run_tests.sh 

On a failed test, the output of `run_tests.sh` should look like:

    [FAIL]: test_1
    [Thu Mar 30 16:28:01 PDT 2017] 0 of 1 tests passed

On success:

    [PASS]: test_1
    [Thu Mar 30 16:25:57 PDT 2017] 1 of 1 tests passed



One test has been provided as a way to check your formatting and simulate how we will be running tests when you submit your solution. We urge you to write your own additional tests. `test_1` is only intended to alert you if the directory structure or the output for this test is incorrect.

Your submission must pass at least the provided test in order to pass the coding challenge.

## Instructions to submit your solution
* To submit your entry please use the link you received in your coding challenge invite email
* You will only be able to submit through the link one time 
* Do NOT attach a file - we will not admit solutions which are attached files 
* Use the submission box to enter the link to your github repo or bitbucket ONLY
* Link to the specific repo for this project, not your general profile
* Put any comments in the README inside your project repo, not in the submission box
* We are unable to accept coding challenges that are emailed to us 

# FAQ

Here are some common questions we've received. If you have additional questions, please email us at `cc@insightdataengineering.com` and we'll answer your questions as quickly as we can (during PST business hours), and update this FAQ.

### Why are you asking us to assume the data is streaming in when creating the `medianvals_by_zip.txt` file but not when creating the `medianvals_by_date.txt` file? 
As a data engineer, you may want to take into consideration future needs. For instance, the team working on the dashboard may want to re-use the streaming functionality used to create `medianvals_by_zip.txt` file in the future to show a running median and total dollar amount of contributions as they arrive in real-time. It might prove useful in assessing the success of a candidate's fundraising efforts at any moment in time. However, because some contributions often arrive later than others and significantly out of order, the final amounts aggregated in `medianvals_by_date.txt` also are useful to the campaign.


### Which Github link should I submit?
You should submit the URL for the top-level root of your repository. For example, this repo would be submitted by copying the URL `https://github.com/InsightDataScience/find-political-donors` into the appropriate field on the application. **Do NOT try to submit your coding challenge using a pull request**, which would make your source code publicly available.

### Do I need a private Github repo?
No, you may use a public repo, there is no need to purchase a private repo. You may also submit a link to a Bitbucket repo if you prefer.

### May I use R, Matlab, or other analytics programming languages to solve the challenge?
It's important that your implementation scales to handle large amounts of data. While many of our Fellows have experience with R and Matlab, applicants have found that these languages are unable to process data in a scalable fashion, so you must consider another language.

### May I use distributed technologies like Hadoop or Spark?
Your code will be tested on a single machine, so using these technologies will negatively impact your solution. We're not testing your knowledge on distributed computing, but rather on computer science fundamentals and software engineering best practices. 

### What sort of system should I use to run my program on (Windows, Linux, Mac)?
You may write your solution on any system, but your source code should be portable and work on all systems. Additionally, your `run.sh` must be able to run on either Unix or Linux, as that's the system that will be used for testing. Linux machines are the industry standard for most data engineering teams, so it is helpful to be familiar with this. If you're currently using Windows, we recommend installing a virtual Unix environment, such as VirtualBox or VMWare, and using that to develop your code. Otherwise, you also could use tools, such as Cygwin or Docker, or a free online IDE such as Cloud9.

### How fast should my program run?
While there are no strict performance guidelines to this coding challenge, we will consider the amount of time your program takes when grading the challenge. Therefore, you should design and develop your program in the optimal way (i.e. think about time and space complexity instead of trying to hit a specific run time value). 

### Can I use pre-built packages, modules, or libraries?
This coding challenge can be completed without any "exotic" packages. While you may use publicly available packages, modules, or libraries, you must document any dependencies in your accompanying README file. When we review your submission, we will download these libraries and attempt to run your program. If you do use a package, you should always ensure that the module you're using works efficiently for the specific use-case in the challenge, since many libraries are not designed for large amounts of data.

### Will you email me if my code doesn't run?
Unfortunately, we receive hundreds of submissions in a very short time and are unable to email individuals if their code doesn't compile or run. This is why it's so important to document any dependencies you have, as described in the previous question. We will do everything we can to properly test your code, but this requires good documentation. More so, we have provided a test suite so you can confirm that your directory structure and format are correct.

### Can I use a database engine?
This coding challenge can be completed without the use of a database. However, if you use one, it must be a publicly available one that can be easily installed with minimal configuration.

### Do I need to use multi-threading?
No, your solution doesn't necessarily need to include multi-threading - there are many solutions that don't require multiple threads/cores or any distributed systems, but instead use efficient data structures.

### What should the format of the output be?
In order to be tested correctly, you must use the format described above. You can ensure that you have the correct format by using the testing suite we've included. If you are still unable to get the correct format from the debugging messages in the suite, please email us at `cc@insightdataengineering.com`.

### Should I check if the files in the input directory are text files or non-text files(binary)?
No, for simplicity you may assume that all of the files in the input directory are text files, with the format as described above.

### Can I use an IDE like Eclipse or IntelliJ to write my program?
Yes, you can use whatever tools you want - as long as your `run.sh` script correctly runs the relevant target files and creates the `medianvals_by_zip.txt` and `medianvals_by_date.txt` files in the `output` directory.

### What should be in the input directory?
You can put any text file you want in the directory since our testing suite will replace it. Indeed, using your own input files would be quite useful for testing. The file size limit on Github is 100 MB so you won't be able to include the larger sample input files in your `input` directory.

### How will the coding challenge be evaluated?
Generally, we will evaluate your coding challenge with a testing suite that provides a variety of inputs and checks the corresponding output. This suite will attempt to use your `run.sh` and is fairly tolerant of different runtime environments. Of course, there are many aspects (e.g. clean code, documentation) that cannot be tested by our suite, so each submission will also be reviewed manually by a data engineer.

### How long will it take for me to hear back from you about my submission?
We receive hundreds of submissions and try to evaluate them all in a timely manner. We try to get back to all applicants **within two or three weeks** of submission, but if you have a specific deadline that requires expedited review, please email us at `cc@insightdataengineering.com`.
