<img src="logo.png" width="350" alt="King's College London Health Informatics (kclhi) OHDSI stack">

# Docker configuration :whale2:

## Prerequisites

Before starting, [download and install docker (machine)](https://docs.docker.com/machine/install-machine/).

## Setup

1. Checkout OMOP CDM v5.2.2:

`git clone -b v5.2.2 git@github.com:OHDSI/CommonDataModel.git db/cdm`

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;or v6.0.0:

`git clone -b v6.0.0 git@github.com:OHDSI/CommonDataModel.git db/cdm`

2. Vocabulary

* CDM v5.2.2: Download vocabulary from [Athena](https://athena.ohdsi.org/vocabulary/list) into a folder called `db/vocabulary`. Minimum codesets: SNOMED, ICD9CM, ICD9Proc, CPT4, HCPCS, RxNorm, NDC, Gender, Race, CMS Place of Service, Ethnicity, Currency and RxNorm. Ensure all instructions in the downloaded Athena folder are followed.

* CDM v6.0.0: Repeat as above, but downloading vocabulary to `etl/vocabulary`.

3. Initially run the `broadsea-webtools` service, and obtain a DDL to define the tables of a `db/results` schema.

```
docker-compose up -d broadsea-webtools
curl "http://localhost:8080/WebAPI/ddl/results?dialect=postgresql&vocabSchema=cdm" >> db/results.sql
docker-compose down
```

4. Data

  * CDM v5.2.2: Download the SynPUF 1000 person dataset from [ltscomputingllc](http://www.ltscomputingllc.com/downloads/) into a folder called `db/data`.

  * CDM v6.0.0: Clone the [Synthea](git@github.com:synthetichealth/synthea.git) patient dataset generator into `etl/data` and follow the instructions within the repository to generate a suitable dataset.

5. Build the services (target v6 compose file with `-f docker-compose.v6.yml` for CDM v6.0.0, for this and subsequent docker commands).

```
docker-compose build
```

6. Run the services, to initialise the database, tracking progress.

```
docker-compose up -d db
docker logs -f ohdsi_db_1
```

7. Start the (remaining) broadsea containers, prompting the webtools container to run its migrations against the (now initialised) database:

```
docker-compose up -d
```

8. Enter the `db` container, and execute the post-migration OHDSI script:

```
docker exec -it ohdsi_db_1 /bin/bash
psql ohdsi --user user -f ohdsi.sql
```

9. Restart the broadsea containers, so they can leverage the information provided by the post-migration script.

```
docker restart ohdsi_broadsea-webtools_1
docker restart ohdsi_broadsea-methods-library_1
```

10. Confirm cohorts can be successfully generated at `http://localhost:8080/atlas/#/cohortdefinitions`.

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
