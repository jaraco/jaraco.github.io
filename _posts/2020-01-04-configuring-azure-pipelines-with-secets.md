---
layout: post
title: How to add a (shared) secret to an Azure release pipeline
---

Let's say you have a credential for a service that you want to use across a number of projects, like say a token for PyPI to upload Python build artifacts. How can you configure that in Azure.

With Travis-CI, the process is as simple as `travis encrypt` and pasting the result into the configuration file or `travis env copy` to upload the value directly to Travis.

# Identify or Create an Azure subscription

Unfortunately, I have two Azure subscriptions, and they both seem to be for Visual Studio Enterprise. I can't tell which one I should use.

For now, I'm using the subscription f68cc4ba-7b09-423e-9a0e-68c6fd9d0c11, which seems to be associated with my "Work or School" account, because the [pip-run project](https://dev.azure.com/jaraco/pip-run/) is associated with my "Work or School" account and because that subscription is the one linked to the project by way of Service Principal Authentication.

On the contrary, the subscription db9079df-1ed4-4475-a341-543afc87979d seems to be associated with my "Personal" account, as is the [setuptools project](https://dev.azure.com/Python/setuptools/).


# Create a Key Vault

First, you need a vault in which to store the keys. Before you can create a new resource, you need to have a resource group in a designated location which to store that resource. I picked `eastus2` arbitrarily. You must also name the group. I named it 'main' for now as I have nothing else against which to distinguish this group from another.

```
az group create --name main --location eastus2
```

Now you can create the vault. I named it 'secrets' for lack of a better name.

```
az keyvault create --name secrets --resource-group main
```

> The name 'secrets' is already in use.

So I have to create a keyvault with a name that's unique across all Azure instances? Yuck.

```
pip-run master $ az keyvault create --name secrets007 --resource-group main
```

Seems secrets007 is available.

```
pip-run master $ az keyvault secret set --vault-name secrets007 --name 'PyPI-token' --value @(keyring.get_password('https://upload.pypi.org/legacy/', '__token__'))
{
  "attributes": {
    "created": "2020-01-04T14:28:58+00:00",
    "enabled": true,
    "expires": null,
    "notBefore": null,
    "recoveryLevel": "Purgeable",
    "updated": "2020-01-04T14:28:58+00:00"
  },
  "contentType": null,
  "id": "https://secrets007.vault.azure.net/secrets/PyPI-token/aaff183ed7e44cd8923783f1f29ff9b8",
  "kid": null,
  "managed": null,
  "tags": {
    "file-encoding": "utf-8"
  },
  "value": "pypi-<elided>"
}
```

I tried "PyPI token" for the name, but:

> Parameter 'secret_name' must conform to the following pattern: '^[0-9a-zA-Z-]+$'.


# Link the Variable Group to the Project Library

With the token created, I was then able to configure the Variable group "Azure secrets" in the pip-run Library:

![Create Variable Group](/images/variable-group.png)

I wasn't able to figure out a way to do this from the CLI.


# Use the Variable

At this point, I'm stuck. I don't know how to enable that variable group in the pipeline.
