# Protein Sequence Design using AlphaBind

This repository helps you set up and run [AlphaBind](https://github.com/A-Alpha-Bio/alphabind) on AWS HealthOmics for protein sequence design. Currently, this repository presents an example, which can be modified as needed for your specific use case.

AlphaBind is an ESM2 based protein language model to predict protein-protein interaction, e.g. antibody-antigen binding. The binding affinity prediction is just based on protein sequence information, and no structural information is required. The model can be fine tuned with labeled datasets, and it can be used to guide protein sequence design and optimization. 

## Getting Started

### Step 1: Get your NGC API token and Store it in AWS Secret Manager

Create a NVIDIA [NGC account](https://docs.nvidia.com/ngc/gpu-cloud/ngc-user-guide/index.html) and [generate an API key](https://org.ngc.nvidia.com/setup/api-key) to download BioNeMo model weights, e.g. ESM2nv

Create an [AWS Secrets Manager Secret](https://docs.aws.amazon.com/secretsmanager/latest/userguide/create_secret.html), if you use [AWS CLI](https://aws.amazon.com/cli/), you can run the following script:

```bash
aws secretsmanager create-secret \
    --name <YourSecretName> \
    --description "My NVIDIA NGC credentials." \
    --secret-string "{\"NGC_CLI_API_KEY\":\"<YourAPIKey>\",\"NGC_CLI_ORG\":\"<YourNGCSignUpOrganization>\"}"
```

### Step 2: Deploy the stack to create the container and AWS HealthOmics workflow

You can deploy the stack using the following script:

```bash
bash scripts/deploy.sh   -b "<DeploymentS3BucketName>"   -n "<CloudFormationStackName>"   -r "<AWS Region>" -s "<YourSecretName>"
```

### Step 3: Run a workflow
Create an IAM role to run HealthOmics jobs, and set up value of `$ROLEARN` to this role ARN. Replace `$OUTPUTLOC` with the S3 folder created for output files like `s3://{mybucket}/alphabind/output/`. Also create a new `params.json` to point to the processed binding affinity data to fine tune AlphaBind model, like:
```txt
{
	"input_training_data": "s3://<yours3bucket>/train_data.csv",
	"max_epochs": 10,
	"seed_sequence": "EVQLVESGGGLVQPGGSLRLSCAASGFNIKDTYIHWVRQAPGKGLEWVARIYPTNGYTRYADSVKGRFTISADTSKNTAYLQMNSLRAEDTAVYYCSRWGGDGFYAMDYWGQGTLVTVSSGGGGSGGGGSGGGGSDIQMTQSPSSLSASVGDRVTITCRASQDVNTAVAWYQQKPGKAPKLLIYSASFLYSGVPSRFSGSRSGTDFTLTISSLQPEDFATYYCQQHYTTPPTFGQGTKVEIKR",
	"mutation_start_idx": 98,
	"mutation_end_idx": 107,
	"target_protein_sequence": "ACHQLCARGHCSGPGPTQCVNCSQFLRGQECVEECRVLQGLPREYVNARHCLPCHPECQPQNGSVTCFGPEADQCVACAHYKDPPFCVARCPSGVKPDLSYMPIWKFPDEEGACQPSPIN",
	"num_seeds": 10,
	"num_generations": 20,
	"generator_type": "esm-simultaneous-random"
}
```

You can follow the data preprocessing step in [this example](https://github.com/A-Alpha-Bio/alphabind/blob/main/alphabind/examples/finetuning_and_inference/tutorial_1_finetuning_alphabind.ipynb) to prepare `train_data.csv` file.

You can run the job using [AWS CLI](https://aws.amazon.com/cli/) from the terminal:
```bash
WFID=
ROLEARN=
OUTPUTLOC=

aws omics start-run --workflow-id $WFID --role-arn $ROLEARN --output-uri $OUTPUTLOC --storage-type STATIC --parameters file://./params.json --name alphabindworkflow
```

Or you can navigate to the AWS console to run the job. All results are written to a location defined within `$OUTPUTLOC` above. To get to the root directory of the ouputs, you can use the `GetRun` API, which provides the path as `runOutputUri`. Alternatively, this location is available in the console as well.


