# Windows 11 Custom Deployment Strategy

This note documents a controlled Windows 11 Pro image-customization and deployment workflow created for a fictional business context using NTLite and VMware Workstation.

The objective was not to build a production-scale endpoint-management platform. The lab focused on preparing a repeatable Windows installation image with selected applications, system settings and unattended options, then validating that key configuration choices persisted after installation.

---

## Deployment Objective

The custom image was prepared for the fictional organization **TechFix Solutions**.

The intended endpoint profile included:

- Windows 11 Pro;
- selected productivity-oriented system settings;
- reduced consumer-oriented content;
- preconfigured local-account installation behavior;
- automatic installation of essential applications;
- organization identity applied to the installed operating system;
- a defined power-management profile;
- validation in a clean VMware virtual machine.

The resulting ISO was created as:

```text
Windows 11 TechFix
```

with the label:

```text
TechFixISO
```

---

## 1. Base Image Preparation

The process started from an original Windows 11 Pro image loaded into NTLite.

Using a clean source image reduced ambiguity during validation because the resulting system could be compared against a known starting point rather than an already-modified installation.

The image was then mounted and prepared for configuration.

---

## 2. Application Integration

Two applications were added to the NTLite **Post-Setup** phase:

- 7-Zip;
- Mozilla Firefox.

Both installers were configured with the silent-install parameter:

```text
/S
```

The purpose was to allow the applications to be installed automatically after the first logon without requiring manual interaction.

### Deployment Intent

This approach demonstrates a basic form of application provisioning during operating-system deployment.

It does **not** represent centralized application lifecycle management. The lab did not use Microsoft Intune, Configuration Manager, Group Policy Software Installation, winget automation, or another enterprise software-distribution platform.

---

## 3. Removal of Non-Essential Components

The custom image removed selected applications and components considered unnecessary for the fictional business endpoint.

The report specifically identified consumer-oriented content such as:

- games;
- Xbox components;
- multimedia applications;
- other preinstalled applications without direct business value.

The changes were intentionally kept controlled rather than aggressively debloating the operating system.

### Engineering Consideration

Removing optional software can reduce unnecessary content and simplify the user environment, but aggressive component removal can also affect servicing, dependencies and future Windows updates.

For that reason, the lab treated image modification as a controlled customization task rather than an attempt to minimize Windows as far as technically possible.

---

## 4. Endpoint Configuration

Several Windows settings were configured in the image for a more business-oriented endpoint experience.

These included:

- dark mode;
- visible file extensions;
- reduced recommendations and suggestions;
- File Explorer behavior adjustments;
- reduced Start-menu recommendations;
- **High performance** power plan.

The power plan was later validated on the deployed system rather than assumed to have been applied successfully.

---

## 5. Unattended Installation Configuration

The NTLite **Unattended** configuration was used to modify the out-of-box setup behavior.

The lab enabled:

```text
Skip online account setup (Microsoft Account)
```

This allowed the installation workflow to use a local account rather than requiring an online Microsoft account during setup.

The fictional organization identity was also configured as:

```text
Registered Organization: TechFix Solutions
Registered Owner: TechFix Solutions
```

### Scope

This demonstrates unattended setup configuration for a laboratory endpoint image.

It should not be interpreted as a complete zero-touch provisioning solution. The exercise did not implement Autopilot, Intune enrollment, domain join automation, Entra ID join, certificate enrollment, BitLocker escrow or policy assignment.

---

## 6. ISO Generation

Before processing the image, the pending configuration was reviewed in NTLite.

The final image included the selected:

- component changes;
- system settings;
- power-plan configuration;
- unattended options;
- Post-Setup application installation.

The build process completed successfully and generated the customized Windows 11 ISO.

The resulting image was then used as the installation source for a new VMware Workstation virtual machine.

---

## 7. Deployment Validation

Validation was performed on the installed operating system rather than relying only on NTLite's build-completion message.

This distinction is important: successful image generation proves that the image-building process completed, but it does not prove that every intended setting is effective after Windows installation.

Three key areas were verified.

### 7.1 Application Provisioning

The installed Windows system was checked to confirm that the Post-Setup applications were present.

The validation confirmed installation of:

- 7-Zip;
- Mozilla Firefox.

This demonstrated that the Post-Setup configuration executed successfully during deployment.

---

### 7.2 Power Plan

The active Windows power schemes were queried with:

```cmd
powercfg /L
```

The output showed **High performance** as the active power scheme.

This provided command-line validation that the configured power-management preference persisted into the installed operating system.

---

### 7.3 Registered Organization

The Windows registry was queried directly:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v RegisteredOwner
```

and:

```cmd
reg query "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion" /v RegisteredOrganization
```

The deployed system returned:

```text
RegisteredOwner           REG_SZ    TechFix
RegisteredOrganization    REG_SZ    TechFix Solutions
```

This confirmed that the organization-related values had been applied to the installed endpoint.

---

## Validation Model

The workflow followed this sequence:

```text
Clean Windows image
        ↓
NTLite customization
        ↓
Post-Setup application configuration
        ↓
Unattended setup configuration
        ↓
ISO generation
        ↓
Fresh VMware deployment
        ↓
Installed-system validation
```

The final validation phase was deliberately separated from image creation.

A build being marked **Completed** was not treated as sufficient evidence that the endpoint configuration worked as intended.

---

## What the Lab Demonstrates

The exercise demonstrates practical experience with:

- Windows image customization;
- Windows 11 Pro deployment;
- NTLite;
- unattended setup options;
- silent application installation;
- endpoint configuration;
- VMware-based deployment testing;
- command-line validation;
- Windows registry validation;
- evidence-driven verification.

---

## What the Lab Does Not Claim

This project does not claim implementation of a production enterprise deployment platform.

The lab did not include:

- Microsoft Intune;
- Windows Autopilot;
- Microsoft Configuration Manager;
- Windows Deployment Services;
- Microsoft Deployment Toolkit;
- Active Directory domain deployment;
- Entra ID enrollment;
- enterprise certificate deployment;
- centralized patch management;
- production security baselines;
- large-scale device lifecycle management.

The project should therefore be read as a **custom Windows endpoint deployment laboratory**, not as evidence of a completed enterprise endpoint-management rollout.

---

## Key Engineering Principle

Image creation and endpoint validation are separate stages.

A customized ISO is only useful if the installed system behaves as intended.

For this reason, the lab validated selected outcomes directly on the deployed Windows 11 endpoint instead of treating successful ISO generation as final proof of success.
