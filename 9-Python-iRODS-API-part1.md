**Authors**
- Arthur Newton 
- Christine Staiger 
- Claudia Behnke (SURF)
- Claudio Cacciari (SURF)
- Maithili Kalamkar Stam (SURF)

**License**
Copyright 2022 SURF BV

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License.

## Goal
You will learn how to interact with iRODS via the python API. In this module we will explore the API in interactive mode. The exercises you will perform here will be similar to the one already done with icommands. You will:

- Explore the python API to iRODS in ipython
- Add and edit metadata
- Query for data using user defined metadata
- Streaming data

## 1. Explore the python API to iRODS in ipython

### 1.1 Set up

If you have followed the earlier modules in this training, you should have performed the following steps already 

- Logged in the Lisa compute cluster
- Cloned the github repository on the Lisa compute cluster (https://github.com/ccacciari/iRODS-RDM-HPC-course.git)

On the login node of the Lisa compute cluster, you have access to *ipython* interpreter, the python package *irods* and you will install [python-irodsclient](https://github.com/irods/python-irodsclient). The login node behaves like any compute node in a cluster, i.e. when your code works here, it will also work on the compute nodes. Later on you will create a script that calls the workflow and runs it on several remote worker nodes without any direct interaction of us.

Install the *irods-pythonclient*:

```
module load 2020
module load iRODS-iCommands/4.3.0
pip3 install python-irodsclient
```

Start an ipython session (first time doing this could take some time):

```sh
ipython3
```

## 2. Metadata

### 2.1 Connect to iRODS
You authenticate yourself with an iRODS username and password (this is different from your Lisa credentials). The module *getpass* asks for passwords without printing the input on screen. With en encoding function we prevent that the variable contains the plain password when we pass it to the iRODS server.

```py
import getpass
pw = getpass.getpass().encode()
```
Now we can create an iRODS session:
```
from irods.session import iRODSSession
session = iRODSSession(host='rsc-test1.irods.surfsara.nl', port=1247, user='irods-user1', password=pw.decode(), zone='surfZone1')
```

Note that you will need to change the username to the one you are given. Throughout this course we will assume you are `irods-user1`. 

You can test whether we have done everything correctly and have access:

```py
coll = session.collections.get('/surfZone1/home/irods-user1')
print(coll.path)
print(coll.data_objects)
print(coll.subcollections)
```

We will need our home collection more often as reference point, so let us store the collection path:

```py
iHome = coll.path
```

### 2.2 Creating metadata
Working with metadata is not completely intuitive, you need a good understanding of python dictionaries and the iRODS python API classes *dataobject*, *collection*, *iRODSMetaData* and *iRODSMetaCollection*.

Have you already uploaded the file `alice.txt` in a previous module of this training course?  
If not, please upload it:

```py
cd iRODS-RDM-HPC-course
iPath = iHome + '/alice.txt'
session.data_objects.put('alice.txt', iPath)
```

We start slowly with first creating some metadata for our data. 
Currently, our data object does not carry any user-defined metadata:

```py
iPath = iHome + '/alice.txt'
obj = session.data_objects.get(iPath)
print(obj.metadata.items())
```

Create a key, value, unit entry for our data object:

```py
obj.metadata.add('SOURCE', 'python API training', 'version 1')
obj.metadata.add('TYPE', 'test file')
```
If you now print the metadata again, you will see a cryptic list:

```py
print(obj.metadata.items())
```
The list contains two metadata python objects.
To work with the metadata you need to iterate over them and extract the AVU triples:

```py
[(item.name, item.value, item.units) for item in obj.metadata.items()]
```
Metadata can be used to search for your own data but also for data that someone shared with you. You do not need to know the exact iRODS logical path to retrieve the file, you can search for data which is annotated accordingly. We will see that in the next section.

### 3 Streaming data
Streaming data is an alternative to upload large data to iRODS or to accumulate data in a data object over time. First you need to create an empty data object in iRODS before you can stream in the data.

```py
content = 'My contents!'.encode()
obj = session.data_objects.create(iHome + '/stream.txt')
```
This will create a place holder for the data object with no further metadata:

```py
print("Name: ", obj.name)
print("Owner: ", obj.owner_name)
print("Size: ", obj.size)
print("Checksum:", obj.checksum)
print("Create: ", obj.create_time)
print("Modify: ", obj.modify_time)
print("Metadata: ", obj.metadata.items())
```

```
vars(obj)
```
We can now stream in our data into that placeholder

```py
with obj.open('w') as obj_desc:
    obj_desc.write(content)
obj = session.data_objects.get(iHome + '/stream.txt')
```

Now we check the metadata again:

```py
print("Name: ", obj.name)
print("Owner: ", obj.owner_name)
print("Size: ", obj.size)
print("Checksum:", obj.checksum)
print("Create: ", obj.create_time)
print("Modify: ", obj.modify_time)
print("Metadata: ", obj.metadata.items())
```
```
vars(obj)
```
