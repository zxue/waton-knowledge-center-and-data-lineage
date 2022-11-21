# Configure Data Lineage in IBM Cloud Pak for Data

Data lineage is the process of tracking the flow of data over time, providing a clear understanding of where the data originated, how it has changed, and its ultimate destination within the data pipeline. It provides an audit trail for data at a very granular level. Refer to “[What is data lineage](https://www.ibm.com/topics/data-lineage)?” for more details.

When Cloud Pak for Data (CP4D version 4.5.3 or later) is installed and Watson Knowledge Catalog (WKC) is enabled in an OpenShift cluster, the data lineage operator named “MANTA Automated Data Lineage” is installed automatically. 

![MANTA Automated Data Lineage](media/manta-operator.png)

However, the data lineage feature must be configured before it’s available for use in WKC. This document outlines the configuration steps.

## Obtain the Manta License Key

Manta is broadly available in data technologies by companies such as IBM, Google, Amazon, Microsoft, SAP, Snowflake, Tableau, Talend, and Teradata. Refer to [Manta licensing policy](https://getmanta.com/licensing-policy/) for more details. The licenses for data lineage from Manta are available through purchases of cloud services.

The license key is presented in an XML format and saved in a license.key file. This file will be used later when it is applied to Manta containers.
```
<license>
    <version>v4</version>
    <validity>2023-03-14</validity>
    <scripts>3000</scripts>
    <connections>0</connections>
    <type>PARTNER</type>
    <revisions>3</revisions>
    <products>mf</products>
    <products>mf_all</products>
    <products>mr_basic</products>
    <products>mr_exporter</products>
    <products>mr_igc</products>
    <products>mr_importer</products>
    <products>mr_integration</products>
    <products>mr_interpolation</products>
    <products>mr_merger</products>
    <products>mr_openexport</products>
    <products>mr_orchestration</products>
    <products>mr_perspectives</products>
    <products>mr_viewer</products>
    <licensee>IBM internal/demo use only</licensee>
    <hash>P2kr … LnQ==</hash>
</license>
```
## Enable Manta Automated Data Lineage in OpenShift 

Enabling the lineage feature of MANTA Automated Data Lineage is an optional step and is only needed if you want to import lineage information in addition to basic metadata.

To enable Manta and Knowledge Graph, go to custom resources in OpenShift. Search "wkc", and open it. Select the Instances tab, and click on "wkc-cr". Open and modify the Yaml file and change both "enableManta" and "enableKnowledgeGrpha" to "true".

![WKC CR](media/wkc-cr.png)

Alternatively, you can use the oc command to enable Manta and Knowledge Graph.

```
oc project cpd-instance
oc edit wkc wkc-cr
```
It may take 15 minutes or longer to update the custom resource. To check the status, run the following oc command and ensure that all three pods are running and the wkc-cr status is "completed".

```
oc get pod |grep manta
oc get wkc

manta-artemis-ccffd5958-6hls2                                1/1     Running     0                 4d19h
manta-dataflow-6b4f4fb66f-hp8rh                              1/1     Running     1 (6h5m ago)      6h12m
manta-keycloak-6ffc96d7d4-mpk5k                              1/1     Running     0                 8h
NAME     VERSION   RECONCILED   STATUS      AGE
wkc-cr   4.5.3     4.5.3        Completed   4d22h
```

## Apply Manta License Key

You can add, update or view the license key from the OpenShift console. Select Secrets under Workloads from the left side navigation pane. Search for "manta" in all projects or the project where Manta operator is installed, for example, "cpd-instance". Locate and click on "manta-keys". Click "Edit Secret" under Actions. Add a new key-pair or modify an existing one.

![Edit Manta License Key](media/edit-manta-license-key.png)

Alternatively, you can log in to OpenShift and apply the license key using the oc command. See "[Enabling lineage import](https://www.ibm.com/docs/en/cloud-paks/cp-data/4.5.x?topic=administering-enabling-lineage-import)" for more details.

```
oc set data secret/manta-keys -n <namespace> --from-file=license.key=./license.key
```

## Verify Data Lineage

To verify that data lineage is working propertly, add a new asset to a project from the CP4D console. Click on "Metadata import", and select "Get lineage". If the option is grayed out, data lineage is not enabled or not configured propertly.
   
![Get Lineage](media/metadata-import-get-lineage.png)

Alternatively, you can run the oc command to check if Manta data lineage has been eabled. Also, you can vefify if signle sign on or basic authentication has been enabled.

```
oc get wkc wkc-cr -o yaml|grep -i manta

  enableMANTA: true
  metadata_discovery_manta_basic_auth_enabled_value: "true"
  metadata_discovery_manta_use_connection_reference_value: "false"

```

## Use Data Lineage

To use data lineage, you can create a new asset and import metadata with data lineage. Once the import job is completed, which takes several minutes, you can open a dataset and view the data lineage.

In a sample database with tables and stored procedures, you can see two types of data lineage: business data lineage and technical data lineage.

Business Data Lineage

![Business Data Lineage](media/business-data-lineage.png)

Technical Data Lineage

![Technical Data Lineage](media/technical-data-lineage.png)

## Troubleshoot Data Lineage

You can log in to the Manta admin console. If single sign one is not eabled, you will need the admin's password. You can find the basic authenticaiton password from the OpenShift console. 

Select Secrets under Workloads from the left side navigation pane. Search for "manta" or "credentials" in all projects or the project where Manta operator is installed, for example, "cpd-instance". Locate and click on "manta-credentials". Scroll down the page and copy the "Manta_Password".

![Manta Credentials](media/manta-credentials.png)

Log in using the url below. Click on "Configuration" from the top, and select "License" from the left side. 

```
<CP4D url>/manta-admin-gui/app/#/platform/processmanager
```

![Manta License](media/manta-license.png)

