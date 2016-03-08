---
layout: post
title:  将nutch2.3的bin/crawl脚本改写为java类
date:   2016-01-19 21:01:11 +08:00
category: nutch
tags: [nutch, shell]
comments: true
---

* content
{:toc}


nutch1.8以后，以前的主控代码`org.apache.nutch.crawl.Crawl`类没了，只剩下对应的控制脚本`bin/crawl`，感觉在IDEA里面调试不方便，所以我了解了下shell脚本,根据nutch2.3的`bin/crawl`和`bin/nutch`脚本，把`bin/crawl`翻译成了java的Crawl类以便在IDEA里面调试


## 代码设计说明

我参考了nutch1.7的`crawl`类，nutch2.3的`bin/crawl`和`bin/nutch`,尽量按照shell脚本的原组织结构和逻辑进行翻译，有些地方不能直接使用的，就稍作了修改。

- 主要的业务逻辑在`public int run(String[] args)`方法里
- 程序主入口是`main`，调用`ToolRunner.run(NutchConfiguration.create(), new Crawl(), args);`执行上面的`run`方法
- `public void binNutch4j(String jobName,String commandLine,String options)`相当于`bin/crawl`脚本里函数`__bin_nutch`的功能
- `public int runJob(String jobName,String commandLine,String options)`相当于脚本`bin/nutch`的功能，这里没有像脚本中那样用`if-else`，也没有使用`switch-case`,而是采用反射创建相应的job
- `public void preConfig(Configuration conf,String options)`用于根据带`-D`参数 commonOptions等指令设置每个Job的配置项
- `CLASS_MAP`是静态(`static`)属性，一个记录JobName和对应的类名的映射关系的哈希表(`HashMap`)

## gora BUG说明
我之前是在每个job是按照脚本使用batchId参数的，遇到了下面这个问题:

>[Gora MongoDb Exception, can't serialize Utf8](http://stackoverflow.com/questions/30662489/gora-mongodb-exception-cant-serialize-utf8)

貌似是序列化问题，好像gora-0.6版本解决了这个BUG,但我的nutch代码是gora-0.5的，不会升级，所以就简单的把`-batchId`参数去掉，使用`-all`参数就行了，这点在代码里可以看到。

关于升级到gora-0.6,有空再研究好了。

通过这个脚本的改写，我了解了脚本的基本使用，同时对之前看的java反射等知识进行了实践，并对nutch的完整爬取流程、主要控制逻辑有了深刻的印象。主要是前面那个gora的BUG卡了我几天，我还以为自己翻译的有问题，看来调试能力还需要加强。

## java代码

这段代码是翻译nutch2.3的`bin/crawl`和`bin/nutch`脚本

`Crawl`类加到在`org.apache.nutch.crawl`包下，源码如下：

~~~java
package org.apache.nutch.crawl;

/**
 * Created by brianway on 2016/1/19.
 * @author brianway
 * @site brianway.github.io
 * org.apache.nutch.crawl.Crawl;
 */


import org.apache.commons.lang.StringUtils;
import org.apache.hadoop.conf.Configuration;
import org.apache.hadoop.util.Tool;
import org.apache.hadoop.util.ToolRunner;
import org.apache.nutch.fetcher.FetcherJob;
import org.apache.nutch.util.NutchConfiguration;
import org.apache.nutch.util.NutchTool;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;
import java.util.Random;

// Commons Logging imports
//import org.apache.hadoop.fs.*;
//import org.apache.hadoop.mapred.*;
//import org.apache.nutch.util.HadoopFSUtil;
//import org.apache.nutch.util.NutchJob;
//import org.apache.nutch.crawl.InjectorJob;
//import org.apache.nutch.crawl.GeneratorJob;
//import org.apache.nutch.fetcher.FetcherJob;
//import org.apache.nutch.parse.ParserJob;
//import org.apache.nutch.crawl.DbUpdaterJob;
//import org.apache.nutch.indexer.IndexingJob;
//import org.apache.nutch.indexer.solr.SolrDeleteDuplicates;

public class Crawl extends NutchTool implements Tool{
    public static final Logger LOG = LoggerFactory.getLogger(Crawl.class);

    /* Perform complete crawling and indexing (to Solr) given a set of root urls and the -solr
   parameter respectively. More information and Usage parameters can be found below. */
    public static void main(String args[]) throws Exception {
        int res = ToolRunner.run(NutchConfiguration.create(), new Crawl(), args);
        System.exit(res);
    }

    //为了编译过
    @Override
    public Map<String, Object> run(Map<String, Object> args) throws Exception {
        return null;
    }


    @Override
    public int run(String[] args) throws Exception {
        if (args.length < 1) {
            System.out.println
                    ("Usage: Crawl -urls <urlDir> -crawlId <crawlID> -solr <solrURL>  [-threads n] [-depth i] [-topN N]");
            // ("Usage: crawl <seedDir> <crawlID> [<solrUrl>] <numberOfRounds>");
            return -1;
        }

        // ------------check args---------
/*
        //！！由脚本直译的，感觉少参数,所以注释掉，换下面的方式
        String seedDir = args[1];
        String crawlID = args[2];
        String solrUrl=null;
        int limit=1;

        if(args.length-1 == 3){
            limit = Integer.parseInt(args[3]);
        }else if(args.length-1 == 4){
            solrUrl = args[3];
            limit = Integer.parseInt(args[4]);
        }else{
            System.out.println("Unknown # of arguments "+(args.length-1));
            System.out.println
                    ("Usage: crawl <seedDir> <crawlID> [<solrUrl>] <numberOfRounds>");
            return -1;
            //"Usage: Crawl <urlDir> -solr <solrURL> [-dir d] [-threads n] [-depth i] [-topN N]"
            //"Usage: crawl <seedDir> <crawlID> [<solrUrl>] <numberOfRounds>";
        }
*/
        String seedDir = null;
        String crawlID = null;
        String solrUrl=null;
        int limit = 0;
        long topN = Long.MAX_VALUE;
        int threads = getConf().getInt("fetcher.threads.fetch", 10);
        //parameter-format in crawl class is
        // like nutch1.7 "Usage: Crawl <urlDir> -solr <solrURL> [-dir d] [-threads n] [-depth i] [-topN N]"
        //not like nutch2.3 "Usage: crawl <seedDir> <crawlID> [<solrUrl>] <numberOfRounds>";
        for (int i = 0; i < args.length; i++) {
            if ("-urls".equals(args[i])) {
                seedDir = args[++i];
            } else if ("-crawlId".equals(args[i])) {
                crawlID = args[++i];
            } else if ("-threads".equals(args[i])) {
                threads = Integer.parseInt(args[++i]);
            } else if ("-depth".equals(args[i])) {
                limit = Integer.parseInt(args[++i]);
            } else if ("-topN".equals(args[i])) {
                topN =   Long.parseLong(args[++i]);
            } else if ("-solr".equals(args[i])) {
                solrUrl = args[++i];
                i++;
            } else  {
                System.err.println("Unrecognized arg " + args[i]);
                return -1;
            }
        }

        if(StringUtils.isEmpty(seedDir)){
            System.out.println("Missing seedDir : crawl <seedDir> <crawlID> [<solrURL>] <numberOfRounds>");
            return -1;
        }
        if(StringUtils.isEmpty(crawlID)){
            System.out.println("Missing crawlID : crawl <seedDir> <crawlID> [<solrURL>] <numberOfRounds>");
            return -1;
        }
        if(StringUtils.isEmpty(solrUrl)){
            System.out.println("No SOLRURL specified. Skipping indexing.");
        }
        if(limit == 0) {
            System.out.println("Missing numberOfRounds : crawl <seedDir> <crawlID> [<solrURL>] <numberOfRounds>");
            return -1;
        }

        /**
         * MODIFY THE PARAMETERS BELOW TO YOUR NEEDS
         */
        //set the number of slaves nodes
        int numSlaves = 1;
        //and the total number of available tasks
        // sets Hadoop parameter "mapred.reduce.tasks"
        int numTasks = numSlaves<<1;
        // number of urls to fetch in one iteration
        // 250K per task?
        //!!这里使用topN
        long sizeFetchlist = topN;//numSlaves *5;
        // time limit for feching
        int timeLimitFetch=180;
        //Adds <days> to the current time to facilitate
        //crawling urls already fetched sooner then
        //db.default.fetch.interval.
        int addDays=0;

        // note that some of the options listed here could be set in the
        // corresponding hadoop site xml param file
        String commonOptions="-D mapred.reduce.tasks="+numTasks+" -D mapred.child.java.opts=-Xmx1000m -D mapred.reduce.tasks.speculative.execution=false -D mapred.map.tasks.speculative.execution=false -D mapred.compress.map.output=true ";

        preConfig(getConf(),commonOptions);

        //initial injection
        System.out.println("Injecting seed URLs");
        String  inject_args = seedDir+" -crawlId "+crawlID;
        binNutch4j("inject",inject_args,commonOptions);

        for(int a=1;a<=limit;a++){
            //-----------generating-------------
            System.out.println("Generating batchId");
            String batchId = System.currentTimeMillis()+"-"+new Random().nextInt(32767);
            System.out.println("Generating a new fetchlist");
            String  generate_args = "-topN "+ sizeFetchlist +" -noNorm -noFilter -adddays "+addDays+" -crawlId "+crawlID+" -batchId "+batchId;
            //String  generate_options = commonOptions;
            int  res = runJob("generate",generate_args,commonOptions);
            System.out.println("binNutch4j generate "+generate_args);
            if(res==0){

            }else if(res == 1){
                System.out.println("Generate returned 1 (no new segments created)");
                System.out.println("Escaping loop: no more URLs to fetch now");
                break;
            }else{
                System.out.println("Error running:");
                System.out.println("binNutch4j generate "+generate_args);
                System.out.println("Failed with exit value "+res);
                return res;
            }
            //--------fetching-----------
            System.out.println("Fetching : ");
            //String fetch_args = batchId+" -crawlId "+crawlID+" -threads "+threads;
            String fetch_args = "-all"+" -crawlId "+crawlID+" -threads "+threads;
            String  fetch_options = commonOptions+" -D fetcher.timelimit.mins="+timeLimitFetch;
            //10 threads
            binNutch4j("fetch",fetch_args,fetch_options);
            //----------parsing--------------
            // parsing the batch
            //
            if(!getConf().getBoolean(FetcherJob.PARSE_KEY, false)){
                System.out.println("Parsing : ");
                //enable the skipping of records for the parsing so that a dodgy document
                // so that it does not fail the full task
                //String parse_args = batchId+" -crawlId "+crawlID;
                String parse_args = "-all"+" -crawlId "+crawlID;
                String  skipRecordsOptions=" -D mapred.skip.attempts.to.start.skipping=2 -D mapred.skip.map.max.skip.records=1";
                binNutch4j("parse",parse_args,commonOptions+skipRecordsOptions);
            }

            //----------updatedb------------
            // updatedb with this batch
            System.out.println("CrawlDB update for "+crawlID);
           // String updatedb_args = batchId+" -crawlId "+crawlID;
            String updatedb_args = "-all"+" -crawlId "+crawlID;
            binNutch4j("updatedb",updatedb_args,commonOptions);

            if(!StringUtils.isEmpty(solrUrl)){
                System.out.println("Indexing "+ crawlID+ " on SOLR index -> " +solrUrl);
                String index_args = batchId+"  -all -crawlId "+crawlID;
                String  index_options = commonOptions+" -D solr.server.url="+solrUrl;
                binNutch4j("index",index_args,index_options);

                System.out.println("SOLR dedup -> "+solrUrl);
                binNutch4j("solrdedup",solrUrl,commonOptions);

            }else{
                System.out.println("Skipping indexing tasks: no SOLR url provided.");
            }

        }

        return 0;
    }

    /**
     * 相当于bin/crawl的函数__bin_nutch的功能
     * @param jobName job
     * @param commandLine
     */

    public void binNutch4j(String jobName,String commandLine,String options)throws Exception{
        int res = runJob(jobName,commandLine,options);
        if(res!=0) {
            System.out.println("Error running:");
            System.out.println(jobName + " " + commandLine);
            System.out.println("Error running:");
            System.exit(res);
        }
    }

    /**
     * 相当于脚本bin/nutch的功能
     *
     * @param jobName
     * @param commandLine
     * @return
     */
    public int runJob(String jobName,String commandLine,String options)throws Exception{
        //这里为了方便，没有像脚本那样用多个if-elif语句，也没有用switch-case,直接用了反射来完成
        Configuration conf = NutchConfiguration.create();
        if(!StringUtils.isEmpty(options)){
            preConfig(conf,options);
        }
        String[] args =  commandLine.split("\\s+");
        String className = CLASS_MAP.get(jobName);
        Class<?> jobClass  =  Class.forName(className);
        Constructor c = jobClass.getConstructor();
        Tool  job =(Tool) c.newInstance();
        System.out.println("---------------runJob: "+jobClass.getName()+"----------------------");
        return  ToolRunner.run(conf, job, args);
    }


    /**
     * 设置每个job的配置
     * @param conf
     * @param options
     */
    public void preConfig(Configuration conf,String options){
        String [] equations = options.split("\\s*-D\\s+");
        System.out.println("options:"+options);
        // i start from  1 not  0, skip the empty string ""
        for (int i=1;i<equations.length;i++) {
            String equation = equations[i];
            String [] pair = equation.split("=");
            //System.out.println(pair[0]+":"+pair[1]);
            conf.set(pair[0],pair[1]);
            //System.out.println("conf print: "+pair[0]+"  "+conf.get(pair[0]));
        }
    }


    /**
     * the map to store the mapping relations jobName->ClassName
     */
    public static HashMap<String,String> CLASS_MAP = new HashMap<String,String>();

    /**
     * init the CLASS_MAP，refer to "bin/nutch"
     */
    static {
        CLASS_MAP.put("inject","org.apache.nutch.crawl.InjectorJob");
        CLASS_MAP.put("generate","org.apache.nutch.crawl.GeneratorJob");
        CLASS_MAP.put("fetch","org.apache.nutch.fetcher.FetcherJob");
        CLASS_MAP.put("parse","org.apache.nutch.parse.ParserJob");
        CLASS_MAP.put("updatedb","org.apache.nutch.crawl.DbUpdaterJob");
        CLASS_MAP.put("readdb","org.apache.nutch.crawl.WebTableReader");
        CLASS_MAP.put("elasticindex","org.apache.nutch.indexer.elastic.ElasticIndexerJob");
        CLASS_MAP.put("index","org.apache.nutch.indexer.IndexingJob");
        CLASS_MAP.put("solrdedup","org.apache.nutch.indexer.solr.SolrDeleteDuplicates");
    }

}

~~~




## 参考资料

>* [Nutch流程控制源码详解（bin/crawl中文注释版）](http://datahref.com/book/article.php?article=nutch_bin_crawl)
>* [Nutch教程——导入Nutch工程，执行完整爬取](http://datahref.com/book/article.php?article=nutch_run_nutch_in_ide)




----

> 作者[@brianway](http://brianway.github.io/)更多文章：[个人网站](http://brianway.github.io/) `|` [CSDN](http://blog.csdn.net/h3243212/) `|` [oschina](http://my.oschina.net/brianway)


