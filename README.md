<img src="logo.png" width="350" alt="King's College London Health Informatics (kclhi) OHDSI stack">

# Docker configuration :whale2:

## Prerequisites

Before starting, [download and install docker (machine)](https://docs.docker.com/machine/install-machine/).

## Setup

1. Checkout OMOP CDM v5.2.2:

`git clone -b v5.2.2 git@github.com:OHDSI/CommonDataModel.git cdm`

or v6.0:

`git clone -b v6.0.0 git@github.com:OHDSI/CommonDataModel.git cdm`

### CDM v5.2.2

2. Download vocabulary from [Athena](https://athena.ohdsi.org/vocabulary/list) into a folder called `vocabulary`. Minimum codesets: SNOMED, ICD9CM, ICD9Proc, CPT4, HCPCS, RxNorm, NDC, Gender, Race, CMS Place of Service, Ethnicity, Currency and RxNorm. Ensure all instructions in the downloaded Athena folder are followed.

3. Download the SynPUF 1000 person dataset from [ltscomputingllc](http://www.ltscomputingllc.com/downloads/) into a folder called `data`.

4. Initially run the `broadsea-webtools` service, and obtain a DDL to define the tables of a `results` schema.

```
docker-compose up -d broadsea-webtools
docker restart broadsea-webtools
curl "http://localhost:8080/WebAPI/ddl/results?dialect=postgresql&vocabSchema=cdm" > results.sql
docker-compose down -v
```

5. Build the services.

```
docker-compose build
```

6. Run the services, to initialise the database, tracking progress. NB. this is an intensive task, which may require a reset (either indirectly via a restart, or directly by (carefully) removing resources using `docker system prune -a`) before proceeding:

```
docker-compose up -d
docker logs -f ohdsi_db_1
```

7. Restart broadsea containers, prompting the webtools container to run its migrations against the (now initialised) database:

```
docker restart ohdsi_broadsea-webtools_1
docker restart ohdsi_broadsea-methods-library_1
```

8. Enter the `db` container, and execute the post-migration OHDSI script:

```
docker exec -it ohdsi_db_1 /bin/bash
psql omop --user user -f ohdsi.sql
```

9. Restart the broadsea containers again, so they can leverage the information provided by the post-migration script.

10. Open RStudio at `http://localhost:8787` and login with the username 'rstudio' and the password 'mypass'. Execute the following commands to create the remainder of the tables required by ATLAS and the API:

```
library(Achilles)

connectionDetails <- createConnectionDetails(dbms = "postgresql",
                                             server = "<DB container IP>/omop",
                                             user = "user",
                                             password = "password")

achilles(connectionDetails = connectionDetails,
         cdmDatabaseSchema = "cdm",
         resultsDatabaseSchema = "results",
         vocabDatabaseSchema = "cdm",
         sourceName = "OHDSI-CDMV5",
         cdmVersion = "5.2.2",
         numThreads = 1,
         runHeel = FALSE)
```

11. Confirm cohorts can be successfully generated at `http://localhost:8080/atlas/#/cohortdefinitions`.

### CDM v6.0.0

...

## Contributing

Please read [CONTRIBUTING.md](CONTRIBUTING.md) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/martinchapman/nokia-health/tags).

## Authors

[kclhi](https://kclhi.org)

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details.

## Acknowledgments

* [OHDSI Broadsea](https://github.com/OHDSI/Broadsea).
