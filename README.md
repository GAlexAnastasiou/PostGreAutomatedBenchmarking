# PostGreAutomatedBenchmarking
Automates the installation, or upgrade of PostgreSQL, automates the tpch benchmarking between current and previous commit.
# PostGreAutomatedBenchmarking

Greetings,
This script automates the installation of TPCH, PostGreSQL and the benchmarking process.
 
 You have to type the scale factor , the directory name in which you want Postgre and tpch installed , and the name of the newly created database , dedicated to benchmarking of the new commits of PostgreSQL.
 
The results come out in graphs and in the terminal, It shows you the difference of the cold runs , maximum hot run, minimum hot run, and the average of the hot runs.

Each of the 22 benchmarking queries is executed 5 times.

There is also a loop implemented, incase there is no new commit, the script will sleep for 5 mins, check if there is a new commit, if not sleep again, if yes , updates to the new commit and runs the benchmarking process. This loop is added as a comment for the moment, for speed testing reasons.

Version 0.1, first commit. Updates are to follow.
Cheerio.
