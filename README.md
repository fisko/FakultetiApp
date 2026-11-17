# Fakulteti i Informatikës – Aplikacion demo në Azure

ASP.NET Core MVC (.NET 10) me **Azure SQL** (studentët), **PostgreSQL** (njoftimet), **Blob Storage** (dokumentet), **Key Vault** (sekretet), **App Service** (hosting), **Application Insights** (monitorim), **Storage Queue** dhe **Azure Function**.

> **Filloni këtu: [docs/udhezues.md](docs/udhezues.md)** (udhëzues hap pas hapi për fillestarë)
> https://github.com/fisko/FakultetiApp/blob/main/docs/udhezues.md
>
> **Të dhënat e klasës (databazat, fjalëkalimet):** [docs/te-dhenat-e-klases.md](docs/te-dhenat-e-klases.md)
>
> **Mësimi 2 (shkurt):** [docs/mesimi-2-appinsights-function.md](docs/mesimi-2-appinsights-function.md)

## Përmbledhje e hapave për studentin

1. **Fork** këtë repo dhe klonojeni (`gh repo fork fisko/FakultetiApp --clone`).
2. `az login --tenant e7dd866d-a284-43a6-90aa-9aad6684a2f0` dhe `az account set --subscription 9f0f210c-952d-4f87-a34a-0f7c1f6a3dcb`.
3. Web app-i: `./scripts/setup.ps1 -Student emri` (shënoni `APP_NAME`).
4. Key Vault-i juaj: `./scripts/keyvault-student.ps1 ...` (vlerat te [te-dhenat-e-klases.md](docs/te-dhenat-e-klases.md); çelësin e Storage e jep profesori).
5. Profesori krijon lidhjen OIDC dhe ju jep `AZURE_CLIENT_ID`.
6. GitHub secrets: `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID`, `APP_NAME`.
7. **Actions → Deploy në Azure → Run workflow**, pastaj hapni `https://<APP_NAME>.azurewebsites.net`.

Në çdo kohë: `./scripts/kontrollo.ps1 -Student emri` tregon çfarë keni krijuar dhe çfarë mungon.

## Si funksionon
- Connection string-ët ruhen në **Key Vault-in tuaj**; App Service i lexon me Managed Identity përmes `@Microsoft.KeyVault(...)`, pa fjalëkalime në kod.
- GitHub Actions hyn në Azure me **OIDC** (pa fjalëkalim), me një federated credential vetëm për forkun tuaj.

## Për profesorin
Burimet e përbashkëta jetojnë në `rg-shared`: [docs/shared-resources.md](docs/shared-resources.md).
```powershell
./scripts/setup-shared.ps1                         # një herë
./scripts/provision-student.ps1 -Student ardian    # databazat dhe container-i (para mësimit)
./scripts/setup.ps1 -Student ardian                # web app (e bën studenti)
./scripts/keyvault-student.ps1 -Student ardian ... # Key Vault (e bën studenti)
./scripts/new-student-identity.ps1 -Student ardian -GitHubRepo "ardian-gh/FakultetiApp"
./scripts/cleanup.ps1 -Student ardian -StopOnly    # ndalon PostgreSQL
./scripts/cleanup.ps1 -Student ardian              # fshin burimet e studentit
```
Studentët kanë Contributor, Key Vault Administrator dhe User Access Administrator vetëm në `rg-<emri>`; nuk kanë qasje në `rg-shared`.

Kujdes: connection string-ët përdorin admin-in e serverit të përbashkët, kështu që një student teknikisht mund të prekë databazat e të tjerëve. Pranueshme për demo, jo për prodhim.

Kosto: SQL ka kufi falas (10 databaza për abonim), App Service B1 (rreth 13 USD/muaj; F1 falas bllokohet nga kuota CPU), PostgreSQL B1ms paguhet (ndalojeni pas mësimit), Key Vault dhe Storage kushtojnë shumë pak.

Lokalisht: `dotnet run --project src` (kërkon LocalDB dhe PostgreSQL ose vendosni `ConnectionStrings__*`).

> Herën e parë në subscription regjistro `Microsoft.OperationalInsights`, `microsoft.insights` dhe `Microsoft.App` (`az provider register -n <emri>`).

## Mesimi 3 - Terraform

Infrastruktura si kod: https://github.com/fisko/FakultetiApp-Terraform

