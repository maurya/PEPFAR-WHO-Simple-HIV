# PEPFAR/WHO Simple Implementation Guide

Documentation: https://docs.google.com/document/d/1OeqTFl2JllglZ-MFJniKhjzYGCl6hqgLzUOQMKNBqyQ/edit#

See [here](https://drive.google.com/drive/folders/1vTVgdkzLHUQ2bfkuk8ymeHXgxtpC1EMJ?usp=sharing) for other artifacts

Base code retieved from FSH School.

For more information, see the [FSH School Basic Tutorial](https://fshschool.github.io/tutorials/basic).


## Setting up your environment

### STEP 1 - Setting up Java and Maven

alias maven to run with latest jdk for HAPI fhir
```
alias mvn14="JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-14.0.2.jdk/Contents/Home && mvn"
```

### STEP 2 - Clone this repo

```sh
git clone git@github.com:intrahealth/simple-hiv-ig.git
cd PEPFAR-WHO-Simple-HIV
```

Run Publisher and create resources. Resources are put in /output
```sh
# only need first time
bash _updatePublisher
# run every time
bash _genonce.sh
```

### STEP 3 - Configure HAPI FHIR Server to process CQL/Measures.

> Note that the HAPI Docker image doesn't include the feature, the server must be run from the JAR or rebuilt with the JAR and then put into your own container.

Clone hapi-fhir-jpaserver-starter
```
git clone git@github.com:hapifhir/hapi-fhir-jpaserver-starter.git
cd hapi-fhir-jpaserver-starter
```
Uncomment the cql line in src/main/resources/application.yaml so that it is enabled.
```sh
#    auto_create_placeholder_reference_targets: false
    cql_enabled: true
#    default_encoding: JSON
```
Run HAPI

```
mvn14 -Djetty.port=8081 jetty:run
```

### STEP 4 - Configure HAPI FHIR Server to process CQL/Measures.


PUT the Library resources. We use PUT to ensure that the id is always the same and when changes are made they overwrite the resource with those updates.

The FHIR Helpers Library generated from the provided CQL produces an error, so we load that directly after downloading it.

```
wget http://build.fhir.org/ig/cqframework/cqf/Library-FHIRHelpers.json
curl -X PUT -H "Content-Type: application/fhir+json" --data @Library-FHIRHelpers.json http://localhost:8080/fhir/Library/FHIRHelpers
```

```sh
cd output
# library resources
for FILE in FHIRCommon AgeRanges \
HIVSimpleAgeGroup HIVSimpleCondition HIVSimpleDemog HIVSimpleGender HIVSimpleGender2 HIVSimpleTestResult HIVSimpleViralLoad \
; do curl -X PUT -H "Content-Type: application/fhir+json" --data @Library-${FILE}.json http://localhost:8080/fhir/Library/${FILE} ; done
```

PUT the Measure resources
```sh
for FILE in HIVSimpleAgeGroup HIVSimpleCondition HIVSimpleDemog \
HIVSimpleGender HIVSimpleGender2 HIVSimpleGenderCohort HIVSimpleGenderSuppData HIVSimpleGenderSuppDataIndiv \
HIVSimpleGenderSubjectList HIVSimpleTestResult HIVSimpleViralLoad \
; do curl -X PUT -H "Content-Type: application/fhir+json" --data @Measure-${FILE}.json http://localhost:8080/fhir/Measure/${FILE} ; done
```

Run a provided example in the browser
```
http://localhost:8080/fhir/Measure/HIVSimpleGenderCohort/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
http://localhost:8080/fhir/Measure/HIVSimpleGender2/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
http://localhost:8080/fhir/Measure/HIVSimpleGender/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
http://localhost:8080/fhir/Measure/HIVSimpleAgeGroup/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
http://localhost:8080/fhir/Measure/HIVSimpleDemog/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
http://localhost:8080/fhir/Measure/HIVSimpleTestResult/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
http://localhost:8080/fhir/Measure/HIVSimpleCondition/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
http://localhost:8080/fhir/Measure/HIVSimpleViralLoad/$evaluate-measure?periodStart=1970-01-01&periodEnd=2021-01-01
```

## Authoring

Implementation Guides are the FHIR community's supported approach to authorship, creation of Library and Measure resources, versioning. The Atom editor and package for CQL are recommended for iterating on CQL.

* Install the Atom editor and [Atom CQL language package](https://github.com/cqframework/atom_cql_support).
* To test that your setup is working, clone this repo, open a CQL file then make sure you have focus on the CQL file. Then right-click and at the bottom will be the CQL menu to `Execute CQL`. This will bring up a results window as it processes tests.

The folder structure must be set up accordingly (from [Atom CQL language package](https://github.com/cqframework/atom_cql_support)):
```
input/cql
input/tests
input/tests/<cql-library-name>
input/tests/<cql-library-name>/<patient-id>
input/tests/<cql-library-name>/<patient-id>/<resource-type-name>/<resource files> // flexible structure
input/vocabulary/codesystem
input/vocabulary/valueset
```
Observe the above or there will be issues.

For example:
* The CQL files are all under `input/cql`
* Library and Measure templates are under `input/resources`. They must be named and fields match the CQL files.
* The `_genonce` script in Publisher will convert those template Library and Measure resources into full resources that can be PUT on a FHIR server.
* Tests must have the `resource.id` of the resource as a folder name. Thus, there is a separate folder for each Patient bundle.
* The Patient bundles inside the folder can actually be named anything, but they are named here by `resource.id`.
