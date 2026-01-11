Add-Type -AssemblyName System.DirectoryServices

$rootDSE = New-Object System.DirectoryServices.DirectoryEntry("LDAP://RootDSE")
$baseDN  = $rootDSE.defaultNamingContext

# Alle Computer
$searcher = New-Object System.DirectoryServices.DirectorySearcher
$searcher.SearchRoot = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$baseDN")
$searcher.Filter = "(objectCategory=computer)"
$searcher.PageSize = 1000
$searcher.PropertiesToLoad.AddRange(@(
    "name",
    "distinguishedName"
))

$computers = $searcher.FindAll()
$result = @()

foreach ($computer in $computers) {

    $compName = $computer.Properties["name"][0]
    $compDN   = $computer.Properties["distinguishedname"][0]

    # Alle BitLocker Keys unter dem Computer
    $blSearcher = New-Object System.DirectoryServices.DirectorySearcher
    $blSearcher.SearchRoot = New-Object System.DirectoryServices.DirectoryEntry("LDAP://$compDN")
    $blSearcher.Filter = "(objectClass=msFVE-RecoveryInformation)"
    $blSearcher.PropertiesToLoad.AddRange(@(
        "name",
        "msFVE-RecoveryPassword",
        "whenCreated",
        "distinguishedName"
    ))

    foreach ($bl in $blSearcher.FindAll()) {

        $cn = $bl.Properties["name"][0]
        $recoveryId = if ($cn -match "\{([0-9A-Fa-f\-]+)\}") { $matches[1] } else { $null }

        $result += [PSCustomObject]@{
            ComputerName       = $compName
            RecoveryID         = $recoveryId
            RecoveryKey        = $bl.Properties["msfve-recoverypassword"][0]
            Created            = $bl.Properties["whencreated"][0]
        }
    }
}

$result
