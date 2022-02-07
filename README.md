# Compilation
* In ./src directory, we give all modified files and our new-created files (lfh.h/.c, nodeDirectMap.h/.c and query_split.h/.c). To compile our codes, you need to first get the source codes of PostgreSQL 12.3. And then placing the given files into corresponding file folder. All needed file folders are given by Postgres.
* Adding the given files to the postgres project or do corresponding modification to makefile, and compiling the postgres project.
# Reproduction
* We provide two version for reproduction, the first one is docker, you can pull the image by command "docker pull zhaojy20/query_split:v1"
* Our program support original PostgreSQL, optimal baseline we used, and query split with several implementations that we proposed in our paper. To change via them, you can input the embedded command in a session. When you enter thhese commands, the database will return "ERROR: syntax error at or near "switch" at character 1", this is OK. Because PostgreSQL parser cannot identify these code correctly, however, the parameters are actually changed by our embedded code.
* When you install our modifed PostgreSQL, the default query processing mode is orignial PostgreSQL. By embedded command "switch to Postgres;", you can switch the current mode to original PostgreSQL as well.
* By "switch to Optimal;", we switch the execution mode to our baseline. Our baseline obtains the optimal plan by iterating same query many times until the plan is not changed. This mode only support the command decorated by "explain", as we need to record the intermediate execution rersults and record them for later iteration to obtain the optimal plan and itsd execution time. As original "explain" command will finish after optimizer and then output the plan directively, we modified the source code of this part, so both "explain" and "explain(analyze)" will actually execute the plan. The difference between this two commands is that "explain(analyze)" will record the execution time of each step and whether they output actual rows explicitly. In our baseline, an important configuration is the fator that we used to encourage optimizer to try never-met join order. And by "set factor = XX (double);", you can change this factor. If factor < 1, we encourage optimzier to try new join order (we use 0.1 for most queries and 0.5 for some extremely large queries). The less factor is, the longer time is needed to obtain the optimal plan.
* Query split consists of two parts: query splitting algorithm and execution ordr decision. By command "switch to minsubquery;", "switch to relationshipcenter;" and "switch to entitycenter;", you change the query splitting algorithm. By command "switch to oc;"(cost(q)), "switch to or;"(row(q)), "switch to c_r;"(hybrid_row), "switch to c_rsq;"(hybrid_sqrt), "switch to c_rlg;"(hybrid_log) and "switch to global;" (global_sel), you change the execution ordr decision.
* The command "explain" is not supported yet in query split. And query split is not support for non-SPJ query, such as outer join, except etc.
* By command "enable\disable DM;", you allow or disallow the physical opertor "DirectMap", which is an attempt for the improvement of merge join and hash join.

# Potential Bugs
* The local buffer would be released after the end of a session. We recommand execute one query each session for reproducing query split, as the local buffer consumpition is very huge in query split. Executing too many queries in one session may lead to some unfixed bugs. In future, we will improve this part.
* Obviously, our work can be further improved. And there would be many potential bugs in our codes. If you meet these bugs, thank you for contacting us. YYour contact will greatly help us fix these bugs.
