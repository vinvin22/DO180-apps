# Récupération des secrets via CyberArk:

1. Pour pouvoir se connecter et faire des appels webservices avec CyberArk la première étape est de s'authentifier via la mèthode suivante **/POST**
	`https://<CyberArk_Server_Ip>/PasswordVault/API/auth/Cyberark/Logon`
	Le Body de l'appel qu'il faut envoyer est le suivant:
	```
	{
		"username":"<user_name>",
		"password":"<password>",
		"newPassword":"<NewPassword>",
	}
	```
	
	Le retour de cet appel sera comme suit:
	```
	{ 
		"<session token>"
	}
	```
	
2.	Il faut récupérer aprés la liste des accounts via la mèthode suivante **/GET**
	`https://<CyberArk_Server_Ip>/PasswordVault/API/Accounts`
	Le header doit contenir l'authorization avec la valeur du token qu'on a récupérée dans la première étape. Le retour de cet appel sera comme suit:
	```		
	{
	  "id": "string",
	  "name": "string",
	  "address": "string",
	  "userName": "string",
	  "platformId": "string",
	  "safeName": "string",
	  "secretType": "key",
	  "platformAccountProperties": {},
	  "secretManagement": {
		"automaticManagementEnabled": true,
		"manualManagementReason": "string",
		"status": "inProcess",
		"lastModifiedTime": 0,
		"lastReconciledTime": 0,
		"lastVerifiedTime": 0
	  },
	  "remoteMachinesAccess": {
		"remoteMachines": "string",
		"accessRestrictedToRemoteMachines": true
	  },
	  "createdTime": 0
	  "categoryModificationTime": 111111111111111111111
	}
	```

3. Filtrage sur le résultat obtenu selon le champ **userName** *(appName/clientId)* pour récupérer l'accountId de l'application fournie dans le fichier d'entréé *"credentials.json"*.

4. un deuxième web service à appeler **/POST**  se basant sur la valeur de l'account id récupérée dans l'étape précédente 
	`https://<IIS_Server_Ip>/PasswordVault/API/Accounts/{accountId}/Password/Retrieve`
	Le header doit contenir l'authorization avec la valeur du token qu'on a récupérée dans la première étape. Le Body de l'appel qu'il faut envoyer est le suivant:
	```
	{
		reason:"<Reason>",
		TicketingSystemName: "<Ticketing system>",
		TicketId: "<Ticketid>",
		Version: <version number>,
		ActionType: "<action type - show\copy\connect>
		isUse: <true\false>,
		Machine: "<my remote machine address>"
	}
	```
	Le retour de cet appel sera comme suit:
	```		
	"<myPassword>"
	```	



5. Ce service permet d'obtenir le secret pour l'application en cours 

6. Refaire les étapes  **3,4 et 5** pour toutes les clientIds fournies dans le fichier *"credentials.json"*

7. Créer un fichier résultat en json qui va contenir pour chaque cientId la valeur du secret trouvée.

8. Le fichier qu'on doit créér doit respecter le format suivant
	```	
	[
		{	"client_id": "2aze4r2df54azrfszqrzaraz",
		"client_secret": "secret_AMH_1"
		},
		{	"client_id": "2aze4r2df54azrfszqrzara3",
		"client_secret": "secret_AMH_2"
		}
	]
	```	
	
	
# Diagramme de séquence:

sequenceDiagram
CyberArktool->>CyberArk: Logon
CyberArk-->>CyberArktool: Authentification key
CyberArktool->>CyberArk: Get accounts
CyberArk-->>CyberArktool: List of account
loop for every clientId
	CyberArktool->>CyberArk:  Get password value
	CyberArk-->>CyberArktool: Secret value
end
