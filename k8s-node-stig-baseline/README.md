# Kubernetes Node for Prisma Cloud

<b>Kubernetes</b> Version 1.21.x <b>

<b>Kubernetes</b> STIG Automated Compliance Validation Profile works with Prisma Cloud to perform automated compliance checks of <b>Kubernetes</b>.

This automated Security Technical Implementation Guide (STIG) validator was developed to reduce the time it takes to perform a security check based upon STIG Guidance from DISA. These check results should provide information needed to receive a secure authority to operate (ATO) certification for the applicable technology.
MongoDB uses Prisma Cloud, which provides an open source compliance, security and policy testing framework for docker images.

Version 1.0

STIG Automated Compliance Validation Profile works Kubernetes.

## Kubernetes STIG Overview

The <b>Kubernetes</b> STIG (https://public.cyber.mil/stigs/) by the United States Defense Information Systems Agency (DISA) offers a comprehensive compliance guide for the configuration and operation of various technologies.
DISA has created and maintains a set of security guidelines for applications, computer systems or networks connected to the DoD. These guidelines are the primary security standards used by many DoD agencies. In addition to defining security guidelines, the STIG also stipulates how security training should proceed and when security checks should occur. Organizations must stay compliant with these guidelines or they risk having their access to the DoD terminated.

[STIG](https://en.wikipedia.org/wiki/Security_Technical_Implementation_Guide)s are the configuration standards for United States Department of Defense (DoD) Information Assurance (IA) and IA-enabled devices/systems published by the United States Defense Information Systems Agency (DISA). Since 1998, DISA has played a critical role enhancing the security posture of DoD's security systems by providing the STIGs. The STIGs contain technical guidance to "lock down" information systems/software that might otherwise be vulnerable to a malicious computer attack.

The requirements associated with the <b>Kubernetes</b> STIG are derived from the [National Institute of Standards and Technology](https://en.wikipedia.org/wiki/National_Institute_of_Standards_and_Technology) (NIST) [Special Publication (SP) 800-53, Revision 4](https://en.wikipedia.org/wiki/NIST_Special_Publication_800-53) and related documents.

While the Kubernetes STIG automation profile check was developed to provide technical guidance to validate information with security systems such as applications, the guidance applies to all organizations that need to meet internal security as well as compliance standards.

---
## Kubernetes Variation Mitigation

In order to assist in the countless variants/implementations of Kubernetes Node installations, a unique method has been developed to inject key values for the Prisma checks to properly return pass/fail responses. The following will detail how that's done and how an end user can utilize this to setup their checks for Prisma Cloud.

### The Process

During the exec stage of the project's pipeline, a script has been developed to take the root level inputs.yml file, parse the values, and apply those values to the proper variables within each applicable check. This is accomplished by utilizing SED and swaping the strings within the JSON data.

### Setting Up github-ci.yml

In order for the exec stage to execute properly, an additional variable must be defined in the exec stage portion. This variable is named INPUTS_YML and requires the value 'true' or 'false'. If true, the exec stage will execute the process listed above. If false, will skip the variable replacement and push rulesets directly to an instance.

&nbsp;

#### Example github-ci.yml

```yml
exec Kubernetes:
  extends: .ci:stage:exec:prisma
  variables:
    INPUTS_YML: 'true'
```

&nbsp;

### Utilizing inputs.yml

In the root of the project directory ($CI_PROJECT_DIR) lives a yml file led inputs.yml. This file is where all values for the Kubernetes Node checks reside. If a new variable or new/updated value for a variable is needed for the checks to function properly, this file will need to be updated to reflect this.

&nbsp;

#### Example Entry in inputs.yml

```yml
manifests_path:
  description: 'Path to Kubernetes manifest files on the target node'
  type: string
  value: '/etc/kubernetes/manifests'
```

&nbsp;

The first line, **manifests_path**, is the variable name used to identify the variable in both the inputs.yml and the checks themeselves. Meaning, that the process of injecting values into the checks utilized the varaible name to identify the variable that needs to be replaced with a specified value. Users will need to update their checks to use the variable name as it is shown in the inputs.yml.

The fields underneath the variable name help identify what the variable is.

- **Description** - details what the variable is and what it's purpose is.

- **Type** - the data type for the variable's value (usually a string).

- **Value** - the value of the variable. This will vary depending on the unique installation of Kubernetes Nodes.

---
**NOTE**

- Both the description and value fields are surrounded by either single quotes ( ' ) or double quotes ( " ).

- If the value field is a path (directory) DO NOT append the last forward slash ( / ).

---
&nbsp;

#### Example of Check With Variable Name

BASH - Before injection of variable value

```bash
#!/bin/bash

fileCheck ()
{
        configFile=$(ls manifests_path | grep apiserver)
        if [[ -z $configFile ]]
        then
                echo "Kubernetes API Server Manifest file was not found in manifests_path. Check failed."
        exit 1
        fi
}

masterCheck ()
{
        fileCheck

        echo "k8s Component - Kubernetes Master"
        audConf=$(grep -i audit-log-maxsize manifests_path/$configFile | cut -d '=' -f 2)
        if [[ -z $audConf ]]
        then
                echo "audit-log-maxsize is not set in the Kubernetes API Server manifest file. Check failed."
                exit 1
        elif [[ $audConf -lt 100 ]]
        then
                echo "audit-log-maxsize is set to less than '100'. Check failed."
                exit 1
        else
                echo "audit-log-maxsize is set correctly in the Kubernetes API Server manifest file. Check passed."
                exit 0
        fi
}

masterCheck
```

BASH - After injection of variable value

```bash
#!/bin/bash

fileCheck ()
{
        configFile=$(ls /etc/kubernetes/manifests | grep apiserver)
        if [[ -z $configFile ]]
        then
                echo "Kubernetes API Server Manifest file was not found in /etc/kubernetes/manifests. Check failed."
        exit 1
        fi
}

masterCheck ()
{
        fileCheck

        echo "k8s Component - Kubernetes Master"
        audConf=$(grep -i audit-log-maxsize /etc/kubernetes/manifests/$configFile | cut -d '=' -f 2)
        if [[ -z $audConf ]]
        then
                echo "audit-log-maxsize is not set in the Kubernetes API Server manifest file. Check failed."
                exit 1
        elif [[ $audConf -lt 100 ]]
        then
                echo "audit-log-maxsize is set to less than '100'. Check failed."
                exit 1
        else
                echo "audit-log-maxsize is set correctly in the Kubernetes API Server manifest file. Check passed."
                exit 0
        fi
}

masterCheck
```
