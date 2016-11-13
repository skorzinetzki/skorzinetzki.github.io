---
layout: post
title:  "Automatische Kopie einer Datenbank in Microsoft Azure"
date:   2016-11-06 12:25:33 +0100
categories: cloud continuous-delivery deployment
tags: azure powershell database automation infrastructure-as-code
---
Im Rahmen der Migration eines Kunden-Systems in die Azure-Cloud arbeiten wir derzeit daran, die im Hintergrund benötigte Infrastrukturbereitstellung zu automatisieren. Neben den üblichen Systemumgebungen (Entwicklung, Test, Produktion) wird für die Abnahme eine QS-Umgebung benötigt. Diese hat als Anforderung, die Produktionsumgebung soweit möglich nachzustellen. Ein wichtiges Kernelement hierfür ist die Datenbank.

Die Produktionsdatenbank wird bereits in Azure betrieben. Es liegt also nahe, diese Datenbank innerhalb von Azure zu replizieren. Durch das Azure SDK lässt sich Azure unter anderem mittels PowerShell ansprechen. Dies bietet die Möglichkeit, verschiedene wiederkehrende Aufgaben per Skript zu automatisieren.
<!--more-->
Folgende Aufgabenstellung ergibt sich daraus:

* Login
* Prüfung auf vorhandene QS-Datenbank
* Falls vorhanden: Löschen der QS-Datenbank
* Kopie der Produktionsdatenbank in QA-Datenbank
* Anpassen des Abo-Preises

Angelehnt an das von [Microsoft zur Verfügung gestellte Beispielskript][azure-copy-db] habe ich mich der Aufgabe angenommen. Das Beispielskript setzt ein paar Dinge voraus:

* Die Anmeldung geschieht interaktiv
* Die Quell-Datenbank muss existieren (inkl. Server, Ressourcengruppe, …)
* Der Ziel-Server muss existieren (inkl. Ressourcengruppe, …)

Während die letzten beiden Punkte in unserem Kontext gegeben sind, trifft dies für Punkt 1 nicht zu. Das Skript soll in der Continuous Delivery Pipeline integriert werden und somit nicht von interaktiven Benutzereingaben abhängig sein.

Folgende Probleme sind dabei zu Tage getreten.

## Problemstellung: automatischer Login

Alternativ zum Cmdlet `Add-AzureRmAccount`, wie es das Skript vorsieht, bietet das Azure-SDK das Cmdlet `Login-AzureRmAccount` an. Diesem wiederum kann ein `PSCredential`-Objekt übergeben werden, das sich aus Benutzername und Passwort zusammensetzt.

Als Login kam für den Test zunächst ein Microsoft Live Konto in Frage. Ein solcher Login lässt sich allerdings nicht in Kombination mit besagtem Cmdlet nutzen (vgl. [Azure PowerShell Issue][azure-powershell-issue]). Deswegen wurde im Azure Active Directory des Accounts ein neuer AD-Benutzer angelegt und mit ausreichenden Rechten versehen:

* Login mit dem Microsoft-Konto
* Service-Seite von Azure Active Directory öffnen und dem Link „Klassisches Portal“ folgen
* Über das klassische Portal einen neuen Benutzer anlegen
* Zurück im neuen Portal die Service-Seite der Abonnements öffnen
* Im jeweiligen Abonnement auf Zugriffssteuerung (IAM) und dort den neu angelegten Benutzer hinzufügen
* Jetzt sollte der Login mit dem neuen Benutzer funktionieren und dieser die betroffenen Services und Ressourcengruppen einsehen können.

{% highlight powershell %}
$azureAccountName ="{USERNAME}"
$azurePassword = ConvertTo-SecureString "{PASSWORD}" -AsPlainText -Force
$psCredential = New-Object System.Management.Automation.PSCredential($azureAccountName, $azurePassword)
Login-AzureRmAccount -Credential $psCredential
{% endhighlight %}

## Problemstellung: richtiges Abonnement auswählen

Da einem Account mehrere Abonnements zugewiesen sein können, muss das richtige Abonnement selektiert werden. Die im Beispielskript vorgesehene Variante mittels dem Cmdlet `Set-AzureRmContext` unter alleiniger Angabe der Subscription-Id funktionierte nicht zuverlässig. Alternativ dazu wurde das Abonnement über das Cmdlet `Get-AzureRmSubscription` durch seinen Namen ermittelt und selektiert.

{% highlight powershell %}
Get-AzureRmSubscription –SubscriptionName "{SUBSCRIPTION-NAME}" | Select-AzureRmSubscription
{% endhighlight %}

## Vollständige Lösung

Abschließend das vollständige Skript. Die in geschweiften Klammern gesetzten Werte sollen natürlich durch tatsächliche Werte ersetzt werden.

{% highlight powershell %}
# Sign in to Azure and set the subscription to work with
# ------------------------------------------------------
$azureAccountName ="{USERNAME}"
$azurePassword = ConvertTo-SecureString "{PASSWORD}" -AsPlainText -Force
$psCredential = New-Object System.Management.Automation.PSCredential($azureAccountName, $azurePassword)
Login-AzureRmAccount -Credential $psCredential

Get-AzureRmSubscription –SubscriptionName "{SUBSCRIPTION-NAME}" | Select-AzureRmSubscription

# SQL database source (the existing database to copy)
# ---------------------------------------------------
$sourceDbName = "{DB-NAME}"
$sourceDbServerName = "{DB-SERVER}"
$sourceDbResourceGroupName = "{RESOURCE-GROUP}"

# SQL database copy (the new db to be created)
# --------------------------------------------
$copyDbName = "{DB-NAME}"
$copyDbServerName = "{DB-SERVER}"
$copyDbResourceGroupName = "{RESOURCE-GROUP}"
$copyDbEdition = "{DB-EDITION}"
$copyDbServiceLevel = "{SERVICE-LEVEL}"

# Delete a database, if exists
# ----------------------------
if (Get-AzureRmSqlDatabase -DatabaseName $copyDbName -ServerName $copyDbServerName -ResourceGroupName $copyDbResourceGroupName)
{
  Remove-AzureRmSqlDatabase -Force -DatabaseName $copyDbName -ServerName $copyDbServerName -ResourceGroupName $copyDbResourceGroupName
}

# Copy a database to a different server
# -------------------------------------
New-AzureRmSqlDatabaseCopy -ResourceGroupName $sourceDbResourceGroupName -ServerName $sourceDbServerName -DatabaseName $sourceDbName -CopyResourceGroupName $copyDbResourceGroupName -CopyServerName $copyDbServerName -CopyDatabaseName $copyDbName
Set-AzureRmSqlDatabase -DatabaseName $copyDbName -ServerName $copyDbServerName -ResourceGroupName $copyDbResourceGroupName -Edition $copyDbEdition -RequestedServiceObjectiveName $copyDbServiceLevel
{% endhighlight %}

*Dieser Blogpost wurde ebenfalls auf [Softwareentwicklung Köln Blog][se-koeln-blog] veröffentlicht.*

[azure-copy-db]: https://azure.microsoft.com/en-us/documentation/articles/sql-database-copy-powershell/
[azure-powershell-issue]: https://github.com/Azure/azure-powershell/issues/1309
[se-koeln-blog]: http://www.softwareentwicklung-koeln.de/automatische-kopie-einer-datenbank-in-microsoft-azure/