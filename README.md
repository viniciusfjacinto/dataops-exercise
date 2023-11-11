## Overview

This repository contains resources and scripts for a DataOps exercise focused on creating and inserting car-related data into MongoDB collections. The exercise covers essential steps, including setting up a local MongoDB instance, creating collections, creating an admin user, connecting to the database using Pymongo, inserting data into collections, joining and aggregating data, and exporting to JSON.

## Table of Contents

- [Requirements](#requirements)
- [Installation](#installation)
- [MongoDB Setup](#mongodb-setup)
- [Scripts](#scripts)
- [Aggregations](#aggregations)

## Requirements

Ensure you have the following dependencies installed:

- [MongoDB](https://www.mongodb.com/try/download/community)
- Python libraries: `pandas`, `dotenv`, `pymongo`, `os`

## Installation
MongoDB Installation:
Download and install MongoDB from [here](https://www.mongodb.com/try/download/community).
Follow the installation instructions for your operating system.
Use Compass UI to manage your MongoDB instance.

Python Dependencies:
Install the required Python libraries using the following command:
`pip install -r requirements.txt`

## MongoDB-Setup

Open Mongosh and run the following code for creating a database and new collections for our project:
```
use cars_db
db.createCollection('carros')
db.createCollection('montadoras')
```

Create an Admin user:
```
use admin

db.createUser(
  {
    user: 'admin_dataops',
    pwd: '*******************',
    roles: [
      { role: "userAdminAnyDatabase", db: "admin"},
      { role: "userWriteAnyDatabase", db: "admin"}
    ]
  }
)
```

# Scripts
Connect to the Database Using Pymongo:

Follow [DataOps.ipynb](https://github.com/viniciusfjacinto/dataops-exercise/blob/main/DataOps.ipynb) code instructions.

It's important that you save your user information into environment variables using dotenv library

The script will create two dataframes with the following structure:

1) carros

![image](https://github.com/viniciusfjacinto/dataops-exercise/assets/87664450/5575b1cb-1f59-4875-bc8a-41a0c14ef037)


2) montadoras

![image](https://github.com/viniciusfjacinto/dataops-exercise/assets/87664450/8d1d8d26-48f1-482d-9ec9-5b28b363a818)

Then it wil insert those data into `cars_db.carros` and `cars_db.montadoras`

# Aggregations
In Mongo Compass we will then JOIN carros and montadoras by the column 'Montadora' with the purpose of bringing column 'Pais' to our table.
Next we're going to count how many car models there are by 'Pais'
At the end we will export the resulting data to a .json

Here are the steps:
1) Joining data
```
[
    {
        $lookup: {
            from: "montadoras",
            localField: "Montadora",
            foreignField: "Montadora",
            as: "joined_data"
        }
    },
    {
        $unwind: "$joined_data"
    },
    {
        $project: {
            _id: 1,  
            Carro: 1, 
            Cor: 1,  
            Montadora: 1, 
            Pais: "$joined_data.Pais"  // Include the Pais field from the joined collection
        }
    }
]
```
2) Aggregating by 'Pais'
```

db.carros.aggregate([
  {
    $lookup: {
      from: "montadoras",
      localField: "Montadora",
      foreignField: "Montadora",
      as: "joined_data",
    },
  },
  {
    $unwind: "$joined_data",
  },
  {
    $project: {
      _id: 1,
      Carro: 1,
      Cor: 1,
      Montadora: 1,
      Pais: "$joined_data.Pais", // Include the Pais field from the joined collection
    },
  },
  {
    $group: {
      _id: "$Pais",
      Carros: { $sum: 1 }, // Count the number of documents in each group
    },
  },
  {
    $out: "carros_por_pais", // Output the result to a new collection named 'grouped_result'
  },
])
```

## The last query creates a new collection named `carros_por_pais` which we will export to .json using the 'Export Data' button in Compass

![image](https://github.com/viniciusfjacinto/dataops-exercise/assets/87664450/9f644494-fb23-4d5c-aaf0-7c3013f68904)
