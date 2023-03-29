${spec.metadata.name} Template
---
${spec.metadata.description!?ensure_ends_with(".")}

<#if spec.metadata.googleReleased>
:memo: This is a Google-provided template! Please
check [Provided templates documentation](<#if spec.metadata.documentationLink?has_content>${spec.metadata.documentationLink}<#else>https://cloud.google.com/dataflow/docs/guides/templates/provided-templates</#if>)
on how to use it without having to build from sources.
</#if>

:bulb: This is a generated documentation based
on [Metadata Annotations](https://github.com/GoogleCloudPlatform/DataflowTemplates#metadata-annotations)
. Do not change this file directly.

## Parameters

### Mandatory Parameters

<#list spec.metadata.parameters as parameter><#if !parameter.optional!false>* **${parameter.name}** (${parameter.label}): ${parameter.helpText?ensure_ends_with(".")}
</#if></#list>

### Optional Parameters

<#list spec.metadata.parameters as parameter><#if parameter.optional!false>* **${parameter.name}** (${parameter.label}): ${parameter.helpText?ensure_ends_with(".")}
</#if></#list>

## Getting Started

### Requirements

* Java 11
* Maven
* Valid resources for mandatory parameters.
* [gcloud CLI](https://cloud.google.com/sdk/gcloud), and execution of the
  following commands:
    * `gcloud auth login`
    * `gcloud auth application-default login`

The following instructions use the
[Templates Plugin](https://github.com/GoogleCloudPlatform/DataflowTemplates#templates-plugin)
. Install the plugin with the following command to proceed:

```shell
mvn clean install -pl plugins/templates-maven-plugin -am
```

### Building Template

<#if flex>
This template is a Flex Template, meaning that the pipeline code will be
containerized and the container will be executed on Dataflow. Please
check [Use Flex Templates](https://cloud.google.com/dataflow/docs/guides/templates/using-flex-templates)
and [Configure Flex Templates](https://cloud.google.com/dataflow/docs/guides/templates/configuring-flex-templates)
for more information.
<#else>
This template is a Classic Template, meaning that the pipeline code will be
executed only once and the pipeline will be saved to Google Cloud Storage for
further reuse. Please check [Creating classic Dataflow templates](https://cloud.google.com/dataflow/docs/guides/templates/creating-templates)
and [Running classic templates](https://cloud.google.com/dataflow/docs/guides/templates/running-templates)
for more information.
</#if>

#### Staging the Template

If the plan is to just stage the template (i.e., make it available to use) by
the `gcloud` command or Dataflow "Create job from template" UI,
the `-PtemplatesStage` profile should be used:

```shell
export PROJECT=<my-project>
export BUCKET_NAME=<bucket-name>

mvn clean package -PtemplatesStage  \
-DskipTests \
-DprojectId="$PROJECT" \
-DbucketName="$BUCKET_NAME" \
-DstagePrefix="templates" \
-DtemplateName="${spec.metadata.internalName}" \
<#if flex>
-pl v2/${spec.metadata.module!} \
<#else>
-pl v1 \
</#if>
-am
```

The command should build and save the template to Google Cloud, and then print
the complete location on Cloud Storage:

```
<#if flex>
Flex Template was staged! gs://<bucket-name>/templates/<#if flex>flex/</#if>${spec.metadata.internalName}
<#else>
Classic Template was staged! gs://<bucket-name>/templates/<#if flex>flex/</#if>${spec.metadata.internalName}
</#if>
```

The specific path should be copied as it will be used in the following steps.

#### Running the Template

**Using the staged template**:

You can use the path above run the template (or share with others for execution).

To start a job with that template at any time using `gcloud`, you can use:

```shell
export PROJECT=<my-project>
export BUCKET_NAME=<bucket-name>
export REGION=us-central1
export TEMPLATE_SPEC_GCSPATH="gs://$BUCKET_NAME/templates/<#if flex>flex/</#if>${spec.metadata.internalName}"

### Mandatory
<#list spec.metadata.parameters as parameter><#if !parameter.optional!false>export ${parameter.name?replace('([a-z])([A-Z])', '$1_$2', 'r')?upper_case?replace("-", "_")}=${(parameter.defaultValue?c)!"<${parameter.name}>"}
</#if></#list>

### Optional
<#list spec.metadata.parameters as parameter><#if parameter.optional!false>export ${parameter.name?replace('([a-z])([A-Z])', '$1_$2', 'r')?upper_case?replace("-", "_")}=${(parameter.defaultValue?c)!"<${parameter.name}>"}
</#if></#list>

gcloud dataflow <#if flex>flex-template<#else>jobs</#if> run "${spec.metadata.internalName?lower_case?replace("_", "-")}-job" \
  --project "$PROJECT" \
  --region "$REGION" \
<#if flex>
  --template-file-gcs-location "$TEMPLATE_SPEC_GCSPATH" \
<#else>
  --gcs-location "$TEMPLATE_SPEC_GCSPATH" \
</#if>
<#list spec.metadata.parameters as parameter>
  --parameters "${parameter.name}=$${parameter.name?replace('([a-z])([A-Z])', '$1_$2', 'r')?upper_case?replace("-", "_")}" <#sep>\</#sep>
</#list>
```

For more information about the command, please check:
<#if flex>
https://cloud.google.com/sdk/gcloud/reference/dataflow/flex-template/run
<#else>
https://cloud.google.com/sdk/gcloud/reference/dataflow/jobs/run
</#if>


**Using the plugin**:

Instead of just generating the template in the folder, it is possible to stage
and run the template in a single command. This may be useful for testing when
changing the templates.

```shell
export PROJECT=<my-project>
export BUCKET_NAME=<bucket-name>
export REGION=us-central1

### Mandatory
<#list spec.metadata.parameters as parameter><#if !parameter.optional!false>export ${parameter.name?replace('([a-z])([A-Z])', '$1_$2', 'r')?upper_case?replace("-", "_")}=${(parameter.defaultValue?c)!"<${parameter.name}>"}
</#if></#list>

### Optional
<#list spec.metadata.parameters as parameter><#if parameter.optional!false>export ${parameter.name?replace('([a-z])([A-Z])', '$1_$2', 'r')?upper_case?replace("-", "_")}=${(parameter.defaultValue?c)!"<${parameter.name}>"}
</#if></#list>

mvn clean package -PtemplatesRun \
-DskipTests \
-DprojectId="$PROJECT" \
-DbucketName="$BUCKET_NAME" \
-Dregion="$REGION" \
-DjobName="${spec.metadata.internalName?lower_case?replace("_", "-")}-job" \
-DtemplateName="${spec.metadata.internalName}" \
-Dparameters="<#list spec.metadata.parameters as parameter>${parameter.name}=$${parameter.name?replace('([a-z])([A-Z])', '$1_$2', 'r')?upper_case?replace("-", "_")}<#sep>,</#sep></#list>" \
<#if flex>
-pl v2/${spec.metadata.module!} \
<#else>
-pl v1 \
</#if>
-am
```