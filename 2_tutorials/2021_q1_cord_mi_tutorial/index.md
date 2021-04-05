---
layout: default
parent: tutorials
title: CORD-MI tutorial (2021 Q1)
nav_order: 3
permalink: /tutorials/2021_q1_cord_mi_tutorial
---


<details markdown="block">
  <summary>
    table of contents
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>


# welcome

welcome to the CORD-MI tutorial (2021 Q1)

this tutorial was designed for the CORD-MI projectathon taking place from March 9 to March 18, 2021.

in case you have any questions, need help, or want to give feedback, please contact [support@fdrtd.com](mailto:support@fdrtd.com)


## overview

### learning goals

in this 60-minute tutorial, you are going to learn:
1. to connect to public servers
2. to set up your local client
3. to set up three parties (locally)
4. to do basic calculations
5. to work with tabular data
6. to set up three parties (distributed)
7. to solve the CORD-MI homework

### metadata

* **level:** beginner to intermediate
* **natural language:** English
* **coding language:** Python
* **operating system:** OS independent
* **domain:** healthcare
* **intended audience:** participants of the [Medical Informatics Initiative](https://www.medizininformatik-initiative.de/en/start)

### prequisites

you need
* basic understanding of the Python programming language

your system needs
* Python 3.x
* internet connectivity


***

# 1. connect to public servers

Federated Secure Computing (FSC) is a client/server architecture to run privacy-preserving computation (PPC) protocols.
in a production environment, every party involved would run their own servers - on premises or in the cloud - to preserve
its custody over their data. as a user, however, you may usually ignore the burden of setting up and running a server.
hence, within the scope of this tutorial, you will be given access to preconfigured public test servers.

**NOTE:** please check the terms of service before accessing any servers.
(if you are a participant of CORD-MI, you are covered and need not check.)

**WARNING:** personal data MUST NOT be uploaded to public test servers under any circumstance. this includes, but is not limited to, electronic health records, patient data (even if sanitized/anonymized) and any data regulated by GDPR or similar legislation. by using the test servers you agree to only upload synthetic test data.

## check if the servers are running

let's check first if we have access to the test servers.

### in your internet browser

simply browse to:

[https://lrz.fdrtd.com/public1/capabilities](https://lrz.fdrtd.com/public1/capabilities)

### from the command line

```console
curl -X GET https://lrz.fdrtd.com/public1/capabilities
```

### expected result

you should see something like

```json
{
  "protocols": {
    "fdrtd-simon": {
      "microservices": {
        [...]
      }
    },
    "fdrtd-sophie": {
      "microservices": {
        [...]
      }
    }
  }
}
```

this is the fdrtd server describing its capabilities.

## learning

Federated Secure Computing is a modular framework built from **protocols** and **microservices**.

**protocols** are implementations of general concepts such as [secure multiparty computation](https://en.wikipedia.org/wiki/Secure_multi-party_computation) or [homomorphic encryption](https://en.wikipedia.org/wiki/Homomorphic_encryption).

**microservices** provide the capability to run particular calculations such as secure sum, private set intersection, and many more.

the implementation of a microservice may vary wildly from protocol to protocol and from vendor to vendor. (the fdrtd platform is open to 3rd party implementations of protocols and microservices.) however, users may expect the exact same behaviour from a microservice completely independent from choice of servers, protocols, and vendors. this is a central feature of **interoperability**.

## understanding the server response

as you can see, the capabilites are grouped by protocol and by service.

for example, fdrtd-sophie is a (S)imple h(O)momor(PHI)c (E)ncryption protocol.

similarly, fdrtd-simon is a (SI)mple (M)ultiparty computati(ON) protocol.

we will be using both during this tutorial, and as you would expect, both offer the microservices **statistics-univariate** and **basic-sum**. these compute the sum of data items, as we need it to solve the CORD-MI homework.

## summary 

fdrtd servers provide multiple protocols and microservices, and you can query their abilities through the enpoint `/capabilities`


***

# 2. set up your local client

## set up a working directory

you should make a directory to hold the source code you are editing.

(optional) you may also want to set up a Python virtual environment.

open up the command line interface and change to your working directory.

## install the fdrtd client

if we wanted to, we would need nothing but [curl](https://curl.se/) to access the servers and use the API. however, it will be much more convenient to use a programming language. any programming language will do, here we will be using [Python](https://www.python.org/). we will also use the fdrtd package which offers a high-level API wrapper for even shorter and much more convenient coding.

### automatic install

the best way to set up the fdrtd client is through the Python package manager:

```console
pip install fdrtd
```

this will install fdrtd in your Python path or in your virtual environment, along with all dependencies.

### manual install

if you cannot use pip or do not want to, you may install the federated client manually:

```console
git clone https://github.com/fdrtd/client-python
```

this will create a directory 'client-python' and download the source files.
the folder will contain a directory 'fdrtd' which you may move to your working directory or make available for import, otherwise.
you will probably also have to resolve any missing dependencies, which at the time of writing are 'requests' and 'urllib3' and their dependencies.

## check the installation

open a Python console and type the following:

```python
import fdrtd
api = fdrtd.MidLevelApi('https://lrz.fdrtd.com/public1')
print(api.get_capabilities())
```

are you seeing the same output as before?

great. you now have a working client-side installation of fdrtd.

## summary

almost always, you will be accessing the fdrtd API through a client-side library. the fdrtd package is a light-weight option.


***

# 3. set up three parties (locally)

## describing the network

we need to tell our servers about the other servers in the network, so they can make contact and compute together.

in our network, we will have three 'nodes' corresponding to the three parties doing computation together.
our network definiton also contains two special nodes, a 'central' one and a 'sync' one:

```python
network_definition = {
    'nodes': [
        'https://lrz.fdrtd.com/public1',
        'https://lrz.fdrtd.com/public2',
        'https://lrz.fdrtd.com/public3'
    ],
    'central': 'https://lrz.fdrtd.com/public4',
    'sync': 'https://lrz.fdrtd.com/public5'
}
```

for secure multiparty computation (SMPC) protocols, the central node is not needed. SMPC protocols they are purely peer-to-peer, so one party equals one server equals one node. however, we might want to try out homomorphic encryption as a faster alternative. for this eventuality, we added the central node just in case.

also, we will be using synchronization microservices to make sure the three parties actually move in step. synchronization could be provided by any of three nodes, but we choose to give it a dedicated server.

**SECURITY WARNING:** for homomorphic encryption protocols, the synchronization server MUST NEVER be the central server itself, or else the central node might learn salts and nonces.

we copy the network definition into a Python file **network_definition.py** in our working directory.

## scripts for Airolo, Bapu, and Cynthia

in the language of the CORD-MI workshop, there are no Alice, Bob, and Charlie; but Airolo, Bapu, and Cynthia.

fist, you will have all three of them on your local machine. later, each of the partner institutions will run one of the scripts.

let's look at Airolo's script first:

```python
# airolo.py

import fdrtd
from network_definition import network_definition

my_network_definition = {
    **network_definition,
    'myself': 0
}
```

we are using the network definition as above, but we are adding an index 'myself'.
this will tell our server which of the three nodes, indexed from 0 to 2, we are.

can you guess what Bapu's script and Cynthia's script look like? you should make three files in your working directory:
**airolo.py**, **bapu.py**, **cynthia.py**.

## connecting to the API

the network definition is all we need to connect to the API and set up a network of the three servers.

```python
# airolo.py

[...]
my_api = fdrtd.HighLevelAPI(my_network_definition)
```

however, because there are multiple groups using the exact same servers, we have to identify ourselves.
we do this by providing a list of strings, we'll call them tokens:


```python
# airolo.py

[...]
my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                            tokens=['CORD-MI workshop', '<something>'])
```

that 'something' should be a string you make up for yourself. later, when you are collaborating with other sites, it is mandatory you all agree on the same tokens. other than that, anything goes.

**RECOMMENDATION**: in production, you should make your token a unique hierarchy, starting with your own namespace and then further specifying.

at this point, you should temporarily add a line

```python
# airolo.py

[...]
print(my_api.get_capabilities())
```

to check if everything is working and you still have connectivity to the server as before?

great! you are ready to do your first distributed calculation.


***

# 4. do basic calculations

## the task

let us start by an addition of the numbers 1 through 10.

we will distribute the numbers 1 through 3 to Airolo, 4 to 6 to Bapu and 7 to 10 to Cynthia.

let us look at the script for Airolo, first.

thanks to the high level API it is very easy to task the calculation in a single line:

```python
print(my_api.compute(protocol='fdrtd-simon',
                     microservice='statistics-univariate',
                     data=[1.0, 2.0, 3.0]))
```

as you can see, the calculation is entirely determined by the protocol (we chose secure multiparty computation), the microservice (univariate statistics will yield, among other things, the zero'th moment, which is just the sum over all data points), and the data.

the three scripts should look like this by now:

<details markdown="block">
    <summary>cynthia.py</summary>

    ```python
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 0
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='statistics-univariate',
                         data=[1.0, 2.0, 3.0]))
    ```
</details>

<details markdown="block">
    <summary>bapu.py</summary>

    ```python
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 1
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='statistics-univariate',
                         data=[4.0, 5.0, 6.0]))
    ```
</details>

<details markdown="block">
    <summary>cynthia.py</summary>

    ```python
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 2
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='statistics-univariate',
                         data=[7.0, 8.0, 9.0, 10.0]))
    ```
</details>

except for the two lines with the node index 'myself' and the data, the scripts are identical. you should save them like this in your working directory.

## the execution

now we need to run the three scripts **airolo.py**, **bapu.py**, and **cynthia.py** in parallel.

if you are not using an IDE, the simplest way is to open three terminal windows next to each other and start them separately:

```console
python airolo.py
```

```console
python bapu.py
```

```console
python cynthia.py
```

if for whatever reason you have to work with a single command line, depending on your operating system, there may be ways to start a process and return to the CLI immediately and let the process finish in the background. e.g. in linux:


```console
python airolo.py &
python bapu.py &
python cynthia.py &
```

## the result

if everything is working fine, all three scripts should yield the same output:

```json
{
  "inputs": 3,
  "result": {
    "coefficient_of_variation": 0.5504818825631803,
    "coefficient_of_variation_mle": 0.5222329678670935,
    "geometric_mean": 4.528728688116532,
    "harmonic_mean": 3.4141715214755166,
    "samples": 10.0,
    "sum": 55.0,
    [...]
  }
}
```

you are seeing some different statistics, and in particular, the sum 55.

notice how all three scripts execute and terminate at the same time. basically, the first two are waiting for the data of the last one entering the calculation.

congratulations, you have done the first distributed computation with fdrtd.

## optional

in all three scripts, change 'fdrtd-simon' to 'fdrtd-sophie' and rerun the scripts.

do you notice a difference in execution time?

fdrtd-sophie is a homomorphic encryption protocol, which at the moment has more microservices available than fdrtd-simon. it is also more powerful in case of vertically partitioned data and generally faster and far more energy efficient for crunching numerical data.

be mindful which protocol you choose, and think of the environment.

for the purpose of this tutorial, and to dutifully complete the CORD-MI homework on secure multiparty computation, switch back to 'fdrtd-simon' now.


***

# 5. work with tabular data

in the CORD-MI workshop, we will use secure multiparty computation to consolidate the input from several sites. this input will be provided by a table of the following format:

| Einrichtungsidentifikator | AngabeDiag1 | AngabeDiag2 | AngabeGeschlecht | AngabeAlter | Anzahl |
| ------------------------- | ----------- | ----------- | ---------------- | ----------- | ------ |
| ...                       | ...         | ...         | ...              | ...         | ...    |

and generally we want to sum over the data in the column 'Anzahl'.

for the simplicity of this tutorial, you will simply extract the age column into an array

```python
table = ...  # table is a two_dimensional array
data = [anzahl for (_, _, _, _, _, anzahl) in table[1:]]
```

if the table contains a first row of column headers, or

```python
table = ...  # table is a two_dimensional array
data = [anzahl for (_, _, _, _, _, anzahl) in table]
```

if it contains the raw data.

then provide `data` to `api.compute` as before. in a production setting, you would have more advanced options:

you could upload the table to the server and flag the column to be summed. this is useful if you want to do different calculations on the same table, some of which might use more than one column.

or, you could point the server directly to the source of data, which may be another RESTful API, a SELECT command into a database, or whatever. in this way, you would avoid having to download to the client in the first place (sensitive data client-side is never a good idea).


***

# 6. set up three parties (distributed)

now it is time to involve your partner sites.

at this point your three scripts should look like this (we are using mock up data from homework block C here):

<details markdown="block">
    <summary>airolo.py</summary>

    ```python
    # prepare the data
    
    table = [
        ['Einrichtungsidentifikator', 'AngabeDiag1', 'AngabeDiag2', 'AngabeGeschlecht', 'AngabeAlter', 'Anzahl'],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(11,20]', 2],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(21,30]', 4],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(31,40]', 5],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(41,50]', 2]
    ]
    
    data = [anzahl for (_, _, _, _, _, anzahl) in table[1:]]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 0
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='statistics-univariate',
                         data=data))
    ```
</details>

<details markdown="block">
    <summary>bapu.py</summary>

    ```python
    # prepare the data
    
    table = [
        ['Einrichtungsidentifikator', 'AngabeDiag1', 'AngabeDiag2', 'AngabeGeschlecht', 'AngabeAlter', 'Anzahl'],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(11,20]', 2],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(21,30]', 2],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(31,40]', 2],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(41,50]', 2]
    ]
    
    data = [anzahl for (_, _, _, _, _, anzahl) in table[1:]]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 1
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='statistics-univariate',
                         data=data))
    ```
</details>

<details markdown="block">
    <summary>cynthia.py</summary>

    ```python
    # prepare the data
    
    table = [
        ['Einrichtungsidentifikator', 'AngabeDiag1', 'AngabeDiag2', 'AngabeGeschlecht', 'AngabeAlter', 'Anzahl'],
        ['260123453-Cynthia', 'E84,-', 'O80', 'f', '(11,20]', 2],
        ['260123453-Cynthia', 'E84,-', 'O80', 'f', '(21,30]', 3],
        ['260123453-Cynthia', 'E84,-', 'O80', 'f', '(31,40]', 2]
    ]
    
    data = [anzahl for (_, _, _, _, _, anzahl) in table[1:]]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 2
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='statistics-univariate',
                         data=data))
    ```
</details>

how do you distribute the calculation among the partner sites?

you have to agree who is to be Airolo, Bapu, or Cynthia.

this is also a good moment to remember that all three sites need to use the same token `<something>`.

then every site runs only the one respective script.

preferrably at the same time, so pick up the phone and talk to your colleagues.

once the third site executed the script, the result will pop up on everyone's screens.

did it work?


# 7. solve the CORD-MI homework

you are now well prepared to solve the CORD-MI homework.

all you have to do is replace the table in the script with the data from the preceding workshop step.

then execute together.

you might want to have a cup of coffee or tea while you are at it.

well done!


## sample solution for exercise block B

for this, we change back from `statistics-univariate` to `basic-sum`. also, we pull the data from an external table (you might want to import a .csv instead, it is totally up to you)

<details markdown="block">
    <summary>network_definition.py</summary>

    ```python
    network_definition = {
        'nodes': [
            'https://lrz.fdrtd.com/public1',
            'https://lrz.fdrtd.com/public2',
            'https://lrz.fdrtd.com/public3'
        ],
        'central': 'https://lrz.fdrtd.com/public4',
        'sync': 'https://lrz.fdrtd.com/public5'
    }
    ```
</details>

<details markdown="block">
    <summary>table.py</summary>

    ```python
    table = [
        ['', 'df_result$institution_id', 'df_result$code', 'df_result$display', 'df_result$gender', 'df_result$age', 'count'], 
        ['1', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(1, 10]', 5], 
        ['2', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(11, 20]', 6], 
        ['3', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(21, 30]', 17], 
        ['4', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(31, 40]', 22], 
        ['5', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(41, 50]', 24], 
        ['6', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(51, 60]', 28], 
        ['7', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(61, 70]', 31], 
        ['8', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(71, 80]', 27], 
        ['9', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(81, 90]', 12], 
        ['10', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(1, 10]', 6], 
        ['11', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(11, 20]', 4], 
        ['12', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(21, 30]', 8], 
        ['13', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(31, 40]', 13], 
        ['14', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(41, 50]', 18], 
        ['15', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(51, 60]', 27], 
        ['16', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(61, 70]', 23], 
        ['17', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(71, 80]', 18], 
        ['18', '260123451-Airolo', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(81, 90]', 7], 
        ['19', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(11, 20]', 4], 
        ['20', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(21, 30]', 2], 
        ['21', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(31, 40]', 5], 
        ['22', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(41, 50]', 4], 
        ['23', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(51, 60]', 2], 
        ['24', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(61, 70]', 2], 
        ['25', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(71, 80]', 8], 
        ['26', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(81, 90]', 7], 
        ['27', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(91, 999]', 2], 
        ['28', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(1, 10]', 1], 
        ['29', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(11, 20]', 4], 
        ['30', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(21, 30]', 3], 
        ['31', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(31, 40]', 2], 
        ['32', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(41, 50]', 1], 
        ['33', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(51, 60]', 6], 
        ['34', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(61, 70]', 7], 
        ['35', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(71, 80]', 10], 
        ['36', '260123451-Airolo', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(81, 90]', 4], 
        ['37', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(1, 10]', 7], 
        ['38', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(11, 20]', 7], 
        ['39', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(21, 30]', 11], 
        ['40', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(31, 40]', 12], 
        ['41', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(41, 50]', 15], 
        ['42', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(51, 60]', 17], 
        ['43', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(61, 70]', 18], 
        ['44', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(71, 80]', 23], 
        ['45', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(81, 90]', 9], 
        ['46', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(91, 999]', 2], 
        ['47', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(1, 10]', 6], 
        ['48', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(11, 20]', 4], 
        ['49', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(21, 30]', 4], 
        ['50', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(31, 40]', 9], 
        ['51', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(41, 50]', 5], 
        ['52', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(51, 60]', 11], 
        ['53', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(61, 70]', 17], 
        ['54', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(71, 80]', 19], 
        ['55', '260123452-Bapu', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(81, 90]', 5], 
        ['56', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(1, 10]', 3], 
        ['57', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(11, 20]', 5], 
        ['58', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(21, 30]', 4], 
        ['59', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(31, 40]', 2], 
        ['60', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(41, 50]', 7], 
        ['61', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(51, 60]', 3], 
        ['62', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(61, 70]', 4], 
        ['63', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(71, 80]', 6], 
        ['64', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(81, 90]', 4], 
        ['65', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(91, 999]', 2], 
        ['66', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(1, 10]', 6], 
        ['67', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(21, 30]', 2], 
        ['68', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(31, 40]', 1], 
        ['69', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(41, 50]', 8], 
        ['70', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(51, 60]', 7], 
        ['71', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(61, 70]', 10], 
        ['72', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(71, 80]', 7], 
        ['73', '260123452-Bapu', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(81, 90]', 3], 
        ['74', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(1, 10]', 4], 
        ['75', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(11, 20]', 5], 
        ['76', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(21, 30]', 11], 
        ['77', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(31, 40]', 8], 
        ['78', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(41, 50]', 11], 
        ['79', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(51, 60]', 12], 
        ['80', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(61, 70]', 12], 
        ['81', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(71, 80]', 22], 
        ['82', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(81, 90]', 13], 
        ['83', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'f', '(91, 999]', 4], 
        ['84', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(1, 10]', 5], 
        ['85', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(11, 20]', 5], 
        ['86', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(21, 30]', 2], 
        ['87', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(31, 40]', 7], 
        ['88', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(41, 50]', 14], 
        ['89', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(51, 60]', 19], 
        ['90', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(61, 70]', 19], 
        ['91', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(71, 80]', 23], 
        ['92', '260123453-Cynthia', 'J12.8 U07.1!', 'Pneumonie durch sonstige Viren', 'm', '(81, 90]', 4], 
        ['93', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(11, 20]', 1], 
        ['94', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(21, 30]', 2], 
        ['95', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(31, 40]', 2], 
        ['96', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(41, 50]', 3], 
        ['97', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(51, 60]', 2], 
        ['98', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(61, 70]', 5], 
        ['99', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'f', '(71, 80]', 3], 
        ['100', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(1, 10]', 3], 
        ['101', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(11, 20]', 2], 
        ['102', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(21, 30]', 3], 
        ['103', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(31, 40]', 1], 
        ['104', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(41, 50]', 4], 
        ['105', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(51, 60]', 4], 
        ['106', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(61, 70]', 10], 
        ['107', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(71, 80]', 9], 
        ['108', '260123453-Cynthia', 'U08.9 U09.9!', 'COVID-19 in der Eigenanamnese,  nicht näher bezeichnet', 'm', '(81, 90]', 3], 
    ]
    ```
</details>

<details markdown="block">
    <summary>airolo.py</summary>

    ```python
    # prepare the data
    
    from table import table
    data = [anzahl for (_, institution, diagnose, _, _, _, anzahl) in table[1:]
            if institution == '260123451-Airolo' and 'J12.8' in diagnose]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 0
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='basic-sum',
                         data=data))
    ```
</details>

<details markdown="block">
    <summary>bapu.py</summary>

    ```python
    # prepare the data
    
    from table import table
    data = [anzahl for (_, institution, diagnose, _, _, _, anzahl) in table[1:]
            if institution == '260123452-Bapu' and 'J12.8' in diagnose]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 1
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='basic-sum',
                         data=data))
    ```
</details>

<details markdown="block">
    <summary>cynthia.py</summary>

    ```python
    # prepare the data
    
    from table import table
    data = [anzahl for (_, institution, diagnose, _, _, _, anzahl) in table[1:]
            if institution == '260123453-Cynthia' and 'J12.8' in diagnose]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 2
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='basic-sum',
                         data=data))
    ```
</details>

complete sample solution for download:

[musterloesungB.tar.gz](musterloesungB.tar.gz)

[musterloesungB.zip](musterloesungB.zip)


## sample solution for exercise block C

for this, we change back from `statistics-univariate` to `basic-sum`. otherwise, the scripts remain unchanged:

<details markdown="block">
    <summary>network_definition.py</summary>

    ```python
    network_definition = {
        'nodes': [
            'https://lrz.fdrtd.com/public1',
            'https://lrz.fdrtd.com/public2',
            'https://lrz.fdrtd.com/public3'
        ],
        'central': 'https://lrz.fdrtd.com/public4',
        'sync': 'https://lrz.fdrtd.com/public5'
    }
    ```
</details>

<details markdown="block">
    <summary>airolo.py</summary>

    ```python
    # prepare the data
    
    table = [
        ['Einrichtungsidentifikator', 'AngabeDiag1', 'AngabeDiag2', 'AngabeGeschlecht', 'AngabeAlter', 'Anzahl'],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(11,20]', 2],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(21,30]', 4],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(31,40]', 5],
        ['260123451-Airolo', 'E84,-', 'O80', 'f', '(41,50]', 2]
    ]
    
    data = [anzahl for (_, _, _, _, _, anzahl) in table[1:]]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 0
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='basic-sum',
                         data=data))
    ```
</details>

<details markdown="block">
    <summary>bapu.py</summary>

    ```python
    # prepare the data
    
    table = [
        ['Einrichtungsidentifikator', 'AngabeDiag1', 'AngabeDiag2', 'AngabeGeschlecht', 'AngabeAlter', 'Anzahl'],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(11,20]', 2],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(21,30]', 2],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(31,40]', 2],
        ['260123452-Bapu', 'E84,-', 'O80', 'f', '(41,50]', 2]
    ]
    
    data = [anzahl for (_, _, _, _, _, anzahl) in table[1:]]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 1
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='basic-sum',
                         data=data))
    ```
</details>

<details markdown="block">
    <summary>cynthia.py</summary>

    ```python
    # prepare the data
    
    table = [
        ['Einrichtungsidentifikator', 'AngabeDiag1', 'AngabeDiag2', 'AngabeGeschlecht', 'AngabeAlter', 'Anzahl'],
        ['260123453-Cynthia', 'E84,-', 'O80', 'f', '(11,20]', 2],
        ['260123453-Cynthia', 'E84,-', 'O80', 'f', '(21,30]', 3],
        ['260123453-Cynthia', 'E84,-', 'O80', 'f', '(31,40]', 2]
    ]
    
    data = [anzahl for (_, _, _, _, _, anzahl) in table[1:]]
    
    
    # do the calculation
    
    import fdrtd
    from network_definition import network_definition
    
    my_network_definition = {
        **network_definition,
        'myself': 2
    }
    
    my_api = fdrtd.HighLevelApi(network_definition=my_network_definition,
                                tokens=['CORD-MI workshop', '<something>'])
    
    print(my_api.compute(protocol='fdrtd-simon',
                         microservice='basic-sum',
                         data=data))
    ```
</details>

complete sample solution for download:

[musterloesungC.tar.gz](musterloesungC.tar.gz)

[musterloesungC.zip](musterloesungC.zip)


# feedback

congratulations for completing the CORD-MI tutorial.

we hope you achieved your learning goals.

please give us feedback: **did the tutorial work? did it meet your expectations? what should be improved?**

we would be grateful if you sent your comments to [support@fdrtd.com](mailto:support@fdrtd.com).

thank you!