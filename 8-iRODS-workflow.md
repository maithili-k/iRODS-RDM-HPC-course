**Authors**
- Claudio Cacciari (SURF)
- Maithili Kalamkar Stam (SURF)

**License**
Copyright 2022 SURF BV

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## Goal
You will learn how to interact with iRODS workflow engine. In this module we will explore iRODS rules. You will:

- execute the word counter program via iRODS
- execute a rule to count words

## 1. Workflow

### 1.1 Set up

If you have followed the earlier modules in this training, you should have performed the following steps already 

- Logged in the Lisa compute cluster
- Cloned the github repository on the Lisa compute cluster (https://github.com/ccacciari/iRODS-RDM-HPC-course.git)

If you have not logged in iRODS yet, please initialize the connection:

```sh
iinit
```

List the content of your home in iRODS:

```sh
ils
```

If the file "alice.txt" is not in your iRODS home, please upload it:

```sh
cd iRODS-RDM-HPC-course
iput alice.txt
```

### 1.2 Executing the word counter program

The iRODS command to execute a rule is called "irule". A rule is a set of instructions interpreted and executed by the iRODS rule engine.  
Some of those instructions are called iRODS microservices. We will use a microservice called "msiExecCmd":  

```sh
irule '{msiExecCmd("wordcounter.sh","alice.txt","null", "null", "null", *out)}' "null" "*out"
```
Does it work?  
Why iRODS does not find the object alice.txt?

Let's try in this way:

```sh
ils -L
```

You should see something like this:

```
/surfZone1/home/irods-user1:
  irods-user1           0 demoResc       187944 2022-06-24.15:12 & alice.txt
        generic    /data/volume_1/irods/Vault/home/irods-user1/alice.txt
```
"/surfZone1/home/irods-user1/alice.txt" is the iRODS path, while "/data/volume_1/irods/Vault/home/irods-user1/alice.txt"
is the file system path. The microservice "msiExecCmd" execute a script that counts words. The script does not understand
the concept of iRODS path. It is a bash script that works at linux level, therefore it is necessary to pass the physical path
of the file as input for the script:

```
irule '{msiExecCmd("wordcounter.sh","/data/volume_1/irods/Vault/home/irods-user1/aliceInWonderland","null", "null", "null", *out)}' "null" "*out"
```

### 1.3 Executing a rule to count words

To avoid the inconvenience of getting the physical path of the data object, it is possible to implement a rule
that wraps the microservice and it is stored on the iRODS server:

```
irule '{countWords(*path, *out)}' *path='/surfZone1/home/claudio/aliceInWonderland' "*out"
```

Now we can use the iRODS path