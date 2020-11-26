- 初始环境：	

``` shell
etc@etc-ruc:~/IR$ ls -al
total 401584
drwxrwxr-x  2 etc etc         4096 11月 25 18:57 .
drwxr-xr-x 31 etc etc         4096 11月 25 18:47 ..
-rw-r--r--  1 etc docker 269219095 11月 25 18:52 metadata.csv
-rwxrwxr-x  1 etc docker     12112 11月 25 18:52 parser
-rw-------  1 etc etc      1142244 11月 23 23:08 qrels-covid_d5_j0.5-5.txt
-rw-r--r--  1 etc etc    140808704 11月 23 05:27 terrier-project-5.3-bin.tar.gz
-rw-r--r--  1 etc etc        18707 11月 26  2020 topics-rnd5.xml

```

​	metadata.csv：待处理的数据集；

​	parser：用于处理数据集解析器；

​	topics-rnd5.xml：查询集；

​	qrels-covid_d5_j0.5-5.txt：文档相关性评定。



- 解析数据集

  ```shell
  etc@etc-ruc:~/IR$ mkdir dataset
  etc@etc-ruc:~/IR$ ./parser metadata.csv 
  etc@etc-ruc:~/IR$ ls -al dataset/ | wc -l
  191179
  etc@etc-ruc:~/IR/dataset$ cd dataset
  etc@etc-ruc:~/IR/dataset$ ls
  ```

  生成了191179个文件，截图如下：

  <img src="/home/etc/.config/Typora/typora-user-images/image-20201125191118141.png" alt="image-20201125191118141" style="zoom:80%;" />

部分文档的截图如下，是标签文件：

![image-20201125191407531](/home/etc/.config/Typora/typora-user-images/image-20201125191407531.png)

- 解压terrier-project-5.3-bin.tar.gz

  ```shell
  etc@etc-ruc:~/IR/dataset$ cd ..
  etc@etc-ruc:~/IR$ tar xf terrier-project-5.3-bin.tar.gz 
  
  total 408144
  drwxrwxr-x  4 etc etc      4096 11月 25 19:15 .
  drwxr-xr-x 31 etc etc      4096 11月 25 19:14 ..
  drwxrwxr-x  2 etc etc   6709248 11月 25 19:14 dataset
  -rw-r--r--  1 etc etc 269219095 11月 25 18:52 metadata.csv
  -rwxrwxr-x  1 etc etc     12112 11月 25 18:52 parser
  -rw-------  1 etc etc   1142244 11月 23 23:08 qrels-covid_d5_j0.5-5.txt
  drwxrwxr-x  9 etc etc      4096 11月 25 19:15 terrier-project-5.3
  -rw-r--r--  1 etc etc 140808704 11月 23 05:27 terrier-project-5.3-bin.tar.gz
  -rw-r--r--  1 etc etc     18707 11月 26  2020 topics-rnd5.xml
  ```

- 设置语料库：

  ```shell
  etc@etc-ruc:~/IR$ cd terrier-project-5.3/
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_setup.sh ../dataset/
  Creating collection.spec file.
  Creating jforests.properties file.
  Creating features.list file.
  Creating logging configuration (logback.xml) file in /home/etc/IR/terrier-project-5.3/etc/
  Creating terrier.properties file.
  ../dataset/zzwpplh7.trec
  ../dataset/zzwpt9vf.trec
  ../dataset/zzxdg5zo.trec
  ../dataset/zzxjv666.trec
  ../dataset/zzygcap5.trec
  ../dataset/zzykyblm.trec
  ../dataset/zzys31e9.trec
  ../dataset/zzz0f41q.trec
  ../dataset/zzzlzo0y.trec
  ../dataset/zzzu24gs.trec
  Updated collection.spec file (191177 lines). Please check that it contains
  all and only all the files to be indexed, or create it manually.
  ```

- 修改etc/terrier.properties配置文件

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ cp etc/terrier.properties etc/terrier.properties_bk
  etc@etc-ruc:~/IR/terrier-project-5.3$ vi etc/terrier.properties
  ```

  修改截图如下：

  ![](/home/etc/.config/Typora/typora-user-images/image-20201125193417049.png)

- 建立索引

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchindexing
  ```

  运行若干年后，如图所示：

  ![](/home/etc/.config/Typora/typora-user-images/image-20201125200543151.png)

- 获取统计信息来验证索引：

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier indexstats
  Collection statistics:
  number of indexed documents: 191176
  size of vocabulary: 156538
  number of tokens: 18575864
  number of pointers: 11822334
  number of fields: 0
  field names: []
  blocks: true
  ```

- 将topics-rnd5.xml作成.trec标签文件，格式如下：

  ![](/home/etc/.config/Typora/typora-user-images/image-20201125204000349.png)

- 检索命令

- （1）DPH：

  ```
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w DPH -t ../search.trec
  ...
  20:33:29.221 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  20:33:29.321 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  20:33:29.322 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 105ms - 1000 results retrieved
  20:33:29.324 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.107
  20:33:29.340 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/DPH_0.res.settings
  20:33:29.343 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.51 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/DPH_0.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/DPH_0.res  -m all_trec >>var/results/DPH_0.eval
  ```

  （2）BM25

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w BM25 -c c:0.4 -t ../search.trec 
  ...
  20:59:40.954 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  20:59:41.050 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  20:59:41.052 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 102ms - 1000 results retrieved
  20:59:41.053 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.103
  20:59:41.063 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/BM25_1.res.settings
  20:59:41.066 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.202 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/BM25_1.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ 
  ```

  （3）BM25F

  ```shell
  #暂不可用
  ```

  （4）TF_IDF

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w TF_IDF -c c:0.4 -t ../search.trec
  ...
  21:14:10.799 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:14:10.897 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:14:10.899 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 208ms - 1000 results retrieved
  21:14:10.900 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.209
  21:14:11.069 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/TF_IDF_2.res.settings
  21:14:11.072 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 30.863 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/TF_IDF_2.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/TF_IDF_2.res -m all_trec >>var/results/TF_IDF_2.eval
  ```

  （5）lemurTF_IDF

  ```shell
  #暂不可用
  ```

  （6）PL2

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w PL2 -c c:0.4 -t ../search.trec 
  ...
  21:23:58.422 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:23:58.539 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:23:58.541 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 191ms - 1000 results retrieved
  21:23:58.542 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.192
  21:23:58.709 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/PL2_3.res.settings
  21:23:58.712 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 30.081 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/PL2_3.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/PL2_3.res -m all_trec >>var/results/PL2_3.eval
  ```

  （7）PL2_F

  ```shell
  #暂不可用
  ```

  （8）Hiemstra_LM

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w Hiemstra_LM -c c:0.4 -t ../search.trec
  ...
  21:36:13.903 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:36:14.000 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:36:14.001 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 110ms - 1000 results retrieved
  21:36:14.003 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.113
  21:36:14.013 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/Hiemstra_LM_4.res.settings
  21:36:14.016 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 7.826 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/Hiemstra_LM_4.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/Hiemstra_LM_4.res -m all_trec >>var/results/Hiemstra_LM_4.eval
  ```

  （9）DHL

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w DLH -c c:0.4 -t ../search.trec
  ...
  21:42:04.399 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:42:04.508 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:42:04.510 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 115ms - 1000 results retrieved
  21:42:04.511 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.116
  21:42:04.521 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/DLH_5.res.settings
  21:42:04.525 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.558 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/DLH_5.res
  
  bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/DLH_5.res -m all_trec >>var/results/DLH_5.eval
  ```

  （10）DHL13

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w DLH13 -c c:0.4 -t ../search.trec
  ...
  21:46:48.458 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:46:48.561 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:46:48.563 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 109ms - 1000 results retrieved
  21:46:48.564 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.11
  21:46:48.574 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/DLH13_6.res.settings
  21:46:48.577 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.296 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/DLH13_6.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/DLH13_6.res -m all_trec >>var/results/DLH13_6.eval
  ```

  （11）In_expB2

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w In_expB2 -c c:0.4 -t ../search.trec
  ...
  21:52:27.802 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:52:27.923 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:52:27.925 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 127ms - 1000 results retrieved
  21:52:27.926 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.128
  21:52:27.936 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/In_expB2_7.res.settings
  21:52:27.940 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.915 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/In_expB2_7.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/In_expB2_7.res -m all_trec >>var/results/In_expB2_7.eval
  ```

  （12）In_expC2

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w In_expC2 -c c:0.4 -t ../search.trec
  ...
  21:55:14.678 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:55:14.794 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:55:14.796 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 121ms - 1000 results retrieved
  21:55:14.797 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.122
  21:55:14.807 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/In_expC2_8.res.settings
  21:55:14.811 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.953 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/In_expC2_8.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/In_expC2_8.res -m all_trec >>var/results/In_expC2_8.eval
  ```

  （13）InL2

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieve -w InL2 -c c:0.4 -t ../search.trec
  ...
  21:57:45.880 [main] INFO  o.t.matching.PostingListManager - Query 50 with 21 terms has 21 posting lists
  21:57:45.985 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  21:57:45.986 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 110ms - 1000 results retrieved
  21:57:45.988 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.112
  21:57:45.998 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/InL2_9.res.settings
  21:57:46.002 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.331 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/InL2_9.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/InL2_9.res -m all_trec >>var/results/InL2_9.eval
  ```

  （14）DFRDependenceScoreModifier

  ```sh
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieval -Dmatching.dsms=DFRDependenceScoreModifier -t ../search.trec
  ...
  22:00:10.039 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  22:00:10.041 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 108ms - 1000 results retrieved
  22:00:10.042 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.109
  22:00:10.052 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/DPH_10.res.settings
  22:00:10.055 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.462 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/DPH_10.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/DPH_10.res -m all_trec >>var/results/DPH_10.eval
  ```

  （15）MRFDependenceScoreModifier

  ```shell
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/terrier batchretrieval -Dmatching.dsms=MRFDependenceScoreModifier -t ../search.trec
  ...
  22:04:20.722 [main] INFO  org.terrier.querying.LocalManager - running process PostFilterProcess
  22:04:20.724 [main] INFO  org.terrier.querying.LocalManager - Finished executing query 50 in 117ms - 1000 results retrieved
  22:04:20.725 [main] INFO  o.t.a.batchquerying.TRECQuerying - Time to process query 50: 0.118
  22:04:20.736 [main] INFO  o.t.a.batchquerying.TRECQuerying - Settings of Terrier written to /home/etc/IR/terrier-project-5.3/var/results/DPH_11.res.settings
  22:04:20.739 [main] INFO  o.t.a.batchquerying.TRECQuerying - Finished topics, executed 50 queries in 4.657 seconds, results written to /home/etc/IR/terrier-project-5.3/var/results/DPH_11.res
  
  etc@etc-ruc:~/IR/terrier-project-5.3$ bin/trec_eval -q ../qrels-covid_d5_j0.5-5.txt var/results/DPH_11.res -m all_trec >>var/results/DPH_11.eval
  ```

  

