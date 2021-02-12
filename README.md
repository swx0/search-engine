# Web Search Engine
## About
A **search engine** is a software system that is designed to carry out web searches (Internet searches), which means to search the World Wide Web in a systematic way for particular information specified in a textual web search query.<br /> 
The 3 primary functions of a search engine include:
- **Crawling** the web
- **Indexing** the results
- **Query processing**

Modern search engines would make use of a wide variety of heuristics(Quality Score etc.) for ranking the search results, in order for the more relevant search results to appear earlier than others. However, for a more simplified search engine, we will be using Nutch, ElasticSearch and Kibana to generate.
## Setup

### General
#### Ubuntu setup
For Windows users: Enable Windows Subsystem for Linux (WSL) and install Ubuntu from the Microsoft Store

In order to access files from Windows, within the Ubuntu bash terminal, edit .bashrc
```
nano ~/.bashrc
```
Add the following line into .bashrc (change the name of the user)
```
alias winhome='cd /mnt/c/Users/<name of user>/'
```
Source for change to be effective
```
source ~/.bashrc
```
#### Java
Check if Java is installed
```
java -version
```
If Java not installed
```
sudo apt-get install openjdk-11-jdk
```
check if JAVA_HOME is set
```
echo $JAVA_HOME
```
If JAVA_HOME is not set
```
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

### Nutch 1.17
*[Official Nutch Tutorial](https://cwiki.apache.org/confluence/display/nutch/NutchTutorial)*

Nutch is an open-source web crawler, it serves as a way to find hyperlinks obtained from browsing a set of given known URLs (known as a seed). Afterwhich, these newly discovered URLs will be updated to the list of pages.

Download apache-nutch-1.17-bin.zip [here](https://archive.apache.org/dist/nutch/1.17/)

To configure the crawler, add the following code between 'configuration' and '/configuration' in *apache-nutch-1.17/conf/nutch-site.xml*

```
 <property>
     <name>http.agent.name</name>
     <value>Crawler</value>
     <description>HTTP 'User-Agent' request header. MUST NOT be empty - 
  please set this to a single word uniquely related to your organization.
  NOTE: You should also check other related properties:
    http.robots.agents
    http.agent.description
    http.agent.url
    http.agent.email
    http.agent.version
  and set their values appropriately.
    </description>
  </property>
  <property>
     <name>plugin.includes</name>
     <value>protocol-http|urlfilter-regex|parse-(html|tika)|index-(basic|anchor)|urlnormalizer-(pass|regex|basic)|scoring-opic|indexer-elastic</value>
  </property>
  <property>
      <name>db.ignore.external.links</name>
      <value>false</value>
      <description>If true, outlinks leading from a page to external hosts or domain
      will be ignored. This is an effective way to limit the crawl to include
      only initially injected hosts or domains, without creating complex URLFilters.
      See 'db.ignore.external.links.mode'.
      </description>
  </property>
  <property>
      <name>elastic.host</name>
      <value>localhost</value>
      <description>The hostname to send documents to using TransportClient.
      Either host and port must be defined or cluster.
      </description>
  </property>
  <property>
      <name>elastic.port</name>
      <value>9300</value>
      <description>
      The port to connect to using TransportClient.
      </description>
  </property>
  <property>
      <name>elastic.cluster</name>
      <value>elasticsearch</value>
      <description>The cluster name to discover. Either host and port must
      be defined.
      </description>
  </property>
  <property>
      <name>elastic.index</name>
      <value>nutch</value>
      <description>
      The name of the elasticsearch index. Will normally be autocreated if it
      doesn't exist.
      </description>
  </property>
```

Create a folder called '**urls**' in the main directory of *apache-nutch-1.17* and create a text document **seed.txt** within that folder. **seed.txt** will contain all the URLs used for the initial crawl(Input at least 1 URL). 

Read the URLs from seed.txt and inject into the created *crawldb*
```
bin/nutch inject crawl/crawldb urls
```

After updating *crawldb*, generate a fetch list and create a segment for fetching
```
bin/nutch generate crawl/crawldb crawl/segments
```
```
s1=`ls -d crawl/segments/2* | tail -1`
echo $s1
```

Run fetch on the segment and parse through
```
bin/nutch fetch $s1
```
```
bin/nutch parse $s1
```
Update *crawldb* from the results
```
bin/nutch updatedb crawl/crawldb $s1
```
Repeat the above procedure for the next segments s2 and s3 to crawl through more URLs
```
bin/nutch generate crawl/crawldb crawl/segments -topN 1000
s2=`ls -d crawl/segments/2* | tail -1`
echo $s2

bin/nutch fetch $s2
bin/nutch parse $s2
bin/nutch updatedb crawl/crawldb $s2
```
```
bin/nutch generate crawl/crawldb crawl/segments -topN 1000
s3=`ls -d crawl/segments/2* | tail -1`
echo $s3

bin/nutch fetch $s3
bin/nutch parse $s3
bin/nutch updatedb crawl/crawldb $s3
```
Invert and index all the links obtained
```
bin/nutch invertlinks crawl/linkdb -dir crawl/segments
```

### ElasticSearch 7.4.2
ElasticSearch is an open-source search and analytics engine built on Apache Lucene.

Download ElasticSearch 7.4.2 (Windows) [here](https://www.elastic.co/downloads/past-releases/elasticsearch-7-4-2)

Start ElasticSearch
```
bin/elasticsearch
```

Test whether ElasticSearch is running by visiting http://localhost:9200.

Index all URLs and contents from Nutch to ElasticSearch
```
bin/nutch index crawl/crawldb/ -linkdb crawl/linkdb/ $s1 -filter -normalize -deleteGone
bin/nutch index crawl/crawldb/ -linkdb crawl/linkdb/ $s2 -filter -normalize -deleteGone
bin/nutch index crawl/crawldb/ -linkdb crawl/linkdb/ $s3 -filter -normalize -deleteGone
```
### Kibana 7.4.2
Kibana offers a user interface to visualise and analyse data from ElasticSearch. 

Download Kibana 7.4.2 (Windows) [here](https://www.elastic.co/downloads/past-releases/kibana-7-4-2)

Start Kibana
```
bin/kibana
```

Test whether Kibana is running by visiting http://localhost:5601. An interface should display.
![kibana interface](https://user-images.githubusercontent.com/76123658/107801258-cf956e00-6d9a-11eb-94a5-9bcdd195b3eb.png)

### Kibana interface
Click on the gear icon found in the left vertical bar

