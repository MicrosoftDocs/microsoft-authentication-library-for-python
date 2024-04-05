# Microsoft Authentication Library (MSAL) for Python Documentation

[![Validate existing links](https://github.com/MicrosoftDocs/microsoft-authentication-library-for-python/actions/workflows/linkvalidator.yml/badge.svg)](https://github.com/MicrosoftDocs/microsoft-authentication-library-for-python/actions/workflows/linkvalidator.yml)

This is the documentation repository for Microsoft Authentication Library (MSAL) for Python. If you are looking for the library, refer to [AzureAD/microsoft-authentication-library-for-python](https://github.com/AzureAD/microsoft-authentication-library-for-python).

## Contributions

Contributions to our documentation are welcome. Make sure to familiarize yourself with the [Microsoft Writing Style Guide](https://learn.microsoft.com/style-guide/welcome/) and the [Contributor Guide](https://learn.microsoft.com/contribute/) before making any changes.

> **Warning**
> **Do not** modify any YML files in the `python/docs-ref-autogen` folder - those are generated automatically from the library source code and any changes will be automatically overwritten the next time the documentation Continuous Integration (CI) job runs. To make changes to any API docs you will need to open a pull request in the [AzureAD/microsoft-authentication-library-for-python](https://github.com/AzureAD/microsoft-authentication-library-for-python) repository.

If you would like to author an entirely new document (such as for a new scenario), make sure to [open an issue](https://github.com/MicrosoftDocs/microsoft-authentication-library-for-python/issues) first. This will allow the engineering team to discuss the proposed changes and ensure that it won't be overwritten by future changes.

## API reference documentation automation

Continuous Integration (CI) jobs for this repository are available in the [automation Azure DevOps instance](https://apidrop.visualstudio.com/Content%20CI/_build?definitionId=5247).

>**Note**
> The changes from the CI job will always be pushed to the `smoke-test` branch first. To propagate them to production, merge the changes into `main`, and then merge `main` to `live`. Ensure that all checks pass before merging changes.

## Microsoft Open Source Code of Conduct

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.
