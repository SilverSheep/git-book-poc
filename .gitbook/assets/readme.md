rta-sdk / [Modules](modules.md)

# RTA-SDK

SDK for custom code execution in RTA-Studio

## Installation and running

Clone the repository and run `npm ci`

## General overview of custom code execution

The custom scripts are available at distinct entry points in `Classification` and `Extraction` pipelines.

The entry point placement in the pipelines can be seen at the [CustomCodeExecution diagram](https://applica.atlassian.net/wiki/spaces/RTA/pages/935591942/Custom+code+execution+in+RTA)

For the script to be usable at a specific entry point it has to be placed in the right folder of the Scripts part of the repository.
- EXTRACTION_AUTOTAGGING

         - script_1
                  - script_1.json
                  - script_1.fixer.ts

- CLASSIFICATION_FEATURES
- EXTRACTION_FEATURE
- EXTRACTION_NORMALIZATION
- EXTRACTION_PREDICTION

The naming convention is standardized, keeping the name of the scripts folder and the script itself and metadata file the same, like in the above example structure. The script file should have extension  `.fixer.ts`.

## Writing the script

Example files are in `test/resources/feature-value-prefixer` folder.

There are two parts of each script:
- typescript file with the class providing transform method for the element we want to transformed by the fixer
- meta json file, specifying:
    - label for the script (name that be shown in the RTA-Studio)
    - description for the script (short explanation of the scripts functions, that should be accessible from RTA-Studio)
    - params - custom parameters available for the user to input in RTA-Studio, that can be then used in the script's transform method, together with their default values

Params have to adhere to appropriate Param.AsObject interface, that can be found in `api-framework-js`. 

The main logic of the element transformation is located in the typescript file.
A class extending `FixerModel` abstract class has to be created. It needs to implement one of the transform metgods from the `FixerInterface`, according to which element we want to modify (`transformFeatures` or `transformNamedEntities`).
The params defined in the json metadata file are available in the transform method, via the params class field. They need to be referenced by their paramId.
Both transform methods have access to the whole document to be processed, via their only argument.

## Verifing script

To verify the script you wrote (if it adheres to the rules of execution) in your project's folder run:
`node node_modules/rta-sdk/lib/verifiers/verifier.js <path_to_script> <path_to_metadata>`

## Documentation
### Syncing with documentation repository

In order to stay up to date with documentation repository add git alias presented below and then use:

`git pull-docs`

just as You would use git pull to stay up to date with documentation. 

#### Alias

Run this command to add alias for pulling documentation
```git config alias.pull-docs '!f() { BRANCH=`git show-branch -a | grep 'release*' | head -n1 | cut -d "[" -f2 | cut -d "]" -f1` && git subtree pull --prefix docs git@ssh.git.applica.ai:documentation/terrarium-sdk-documentation.git "$BRANCH" --squash; }; f'```

#### Hook

You can install pre-commit hook that will remind You of pulling docs.

Just run:

`cat ./hooks/pre-commit.sh >> ./.git/hooks/pre-commit && chmod +x ./.git/hooks/pre-commit`

It will append script to existing hook or create new hook if it didn't exist.

### Generating

To generate documentation, run:

`npm run doc`

Documentation for this project is also a part of `documentation project` for RTA.
This external documentation will get automatically updated with the newest version of auto generated docs from this project, at issuing a new `tag` for it.
If you need to update the documentation without issuing a tag for this repository, you need to push it to the external repo:
`git subtree push --prefix docs git@ssh.git.applica.ai:documentation/terrarium-sdk-documentation.git master`
If you need to update the docs from the external repo:
`git subtree pull --prefix docs git@ssh.git.applica.ai:documentation/terrarium-sdk-documentation.git master --squash`

## Sample script with metadata

### Script in `feature-value-prefixer.fixer.ts`
```typescript
import { FeatureNewInterface } from 'document-service-models/lib/models/interfaces/new/components/feature.new-interface';
import { DocumentNewInterface } from 'document-service-models/lib/models/interfaces/new/document.new-interface';
import { FixerModel } from 'rta-sdk/lib/sdk/fixer.model';
import { updateElement } from 'rta-sdk/lib/sdk/utils/update-element.utils';

export class FeatureValuePrefixer extends FixerModel {
    transformFeatures(document: DocumentNewInterface): FeatureNewInterface[] {
        const features = document.extractedFeatures || [];
        const prefix = this.params['prefix'].stringParamValue?.value || '';
        return features.map((f) => updateElement(f, { value: prefix + 'Fixed value' }));
    }
}
```

### Metadata in `feature-value-prefixer.json`
```json
{
  "label": "Feature Value Prefixer",
  "description": "This is a simple script for testing purposes. It prefixes feature's value with a given profix",
  "params": [
    {"paramId": "prefix",
    "paramLabel": "Prefix",
    "description": "Prefix for feature's value",
    "stringParamValue": {
      "value": "default_value"
      }
    }
  ]
}
```
