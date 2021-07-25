## HYPEROO :: BYPASS LICENSE

---
Author: **Alessandro Sgreccia**\
Date: **25-07-2021**\
Type of bypass: **Code Edited**\
Difficulty: **Easy**\
Patched: **No**\
Tested on: **2.4.16**

---

### Identificazione vulnerabilità
Aprendo Hyperoo ed effettuando la connessione al server, il main form ci mostra il banner di attivazione proponendo diversi tipi di attivazioni.
Qualunque attivazione scegliamo finirà in errore, in quanto i server di Hyperoo sembrano dismessi e non più operativi per permettere al programma di validare la licenza.
Sfogliando la cartella del programma, sembrerebbe non trattarsi di un PE, permettendoci quindi di effettuare la decompilazione del codice.

![Desktop View](https://user-images.githubusercontent.com/13579382/126905362-ab6225e4-4ee1-4a75-9573-ce91b3883ccc.png)

### Decompilazione con dnSpy
Importando l'eseguibile in dnSpy ci saltano subito all'occhio le varie classi del codice. :)

![Desktop View](https://user-images.githubusercontent.com/13579382/126905365-41768180-d838-42b8-a9d6-ad16a537b84f.png)

Dopo un pò di analisi delle varie classi, sotto **Hyperoo.UI.ManagementStudio** troviamo il metodo **ClientWindow** che contiene varie funzioni.
La funzione che andremo a modificare sarà **UpdateClientProperties()**, questo perché dopo aver letto e compreso il codice di svariate funzioni tra cui **ActivateWithCloud()** vediamo questa viene chiamata per aggiornare licenza del programma.

### Patch del codice

```c#
internal void UpdateClientProperties()
{
	try
	{
		base.ClearError();
		this.ClientProperties = this.ClientService.GetClientProperties();
		base.ServiceVersion = new Version(this.ClientProperties.Version);
		this.IsActivated = this.ClientProperties.IsActivated;
		if (this.ClientProperties.IsFreeEdition)
		{
			base.Title = string.Format("Hyperoo Client - FREE Edition - Connected to: {0} (v{1}{2})", this.ServerName, this.ClientProperties.Version, this.ClientProperties.VersionSuffix);
		}
		else if (this.ClientProperties.IsRecurring && this.ClientProperties.IsExpired)
		{
			base.Title = string.Format("{0} - (LICENSE EXPIRED) Connected to: {2} (v{1}{3})", new object[]
			{
				"Hyperoo Client",
				this.ClientProperties.Version,
				this.ServerName,
				this.ClientProperties.VersionSuffix
			});
		}
		else if (this.ClientProperties.IsTrial)
		{
			base.Title = string.Format("{0} - (TRIAL: {2} days remaining) Connected to: {3} (v{1}{4})", new object[]
			{
				"Hyperoo Client",
				"2.4.16",
				(int)this.ClientProperties.TrialPeriodRemaining.TotalDays,
				this.ServerName,
				this.ClientProperties.VersionSuffix
			});
		}
		else
		{
			base.Title = string.Format("{0} - Connected to: {2} (v{1}{3})", new object[]
			{
				"Hyperoo Client",
				this.ClientProperties.Version,
				this.ServerName,
				this.ClientProperties.VersionSuffix
			});
		}
		if (!string.IsNullOrEmpty(this.ClientProperties.LatestVersion))
		{
			base.Title += " *New version available*";
		}
		this.PopulateDynamicBanner();
		if (this.ClientProperties.IsRecurring && this.ClientProperties.IsExpired)
		{
			base.SetError("Your license has expired. \nPlease renew your license using the Hyperoo Cloud Console or activate using another license key.");
		}
	}
	catch (Exception ex)
	{
		ExceptionHandler.HandleException(ex, false);
	}
}
```

Andiamo a modificare il codice in modo che ogni volta che questa funzione viene chiamata, il prodotto verrà riconosciuto come **Attivato**.

```c#
internal void UpdateClientProperties()
{
	try
	{
		base.ClearError();
		this.ClientProperties = this.ClientService.GetClientProperties();
		this.ClientProperties.IsActivated = true;
		this.ClientProperties.IsRecurring = true;
		this.ClientProperties.IsExpired = false;
		base.ServiceVersion = new Version(this.ClientProperties.Version);
		this.IsActivated = true;
		MessageBox.Show("Cracked by Alessandro Sgreccia!");
```
Rechiamoci poi in **Hyperoo.UI.ManagementStudio.ClientWindow**, cerchiamo la funzione **IsActivated** e modifichiamo il codice in:

```c#
public bool IsActivated
		{
			get
			{
				return true;
			}
			set
			{
				base.SetValue(ClientWindow.IsActivatedProperty, true);
			}
		}

```
Nella funzione successiva, andiamo ad editare il valore **this.IsActivated = false** in **true**, nella penultima riga del blocco.

```c#
public ClientWindow(IClientManagementService clientService, string sServerName)
		{
			this.InitializeComponent();
			this.ClientService = clientService;
			ClientWindow.ClientConfig = clientService.GetClientConfiguration();
			ClientWindow.OriginalConfig = ClientWindow.ClientConfig.Copy();
			base.DataContext = ClientWindow.ClientConfig;
			this.ServerName = sServerName;
			this.UpdateClientProperties();
			this.consoleMessageClient = new ConsoleMessageClient(ConsoleMessageDestination.ClientConfig, clientService);
			if (ClientWindow.ClientConfig.Tasks.Count > 0)
			{
				this.SelectedTask = ClientWindow.ClientConfig.Tasks[0];
			}
			this.timer = new Timer(new TimerCallback(this.OnTimer), null, 500, 500);
			this.consoleMessageTimer = new Timer(new TimerCallback(this.OnConsoleMessageTimer), null, 100, 2000);
			if (this.ClientProperties != null && !this.ClientProperties.IsActivated)
			{
				this.IsActivated = true;
			}
			this.NavigateToTaskList();
		}
```

### Compilazione 
Assembliamo quindi l'eseguibile, andando a sovrascrivere l'originale.
Eseguendolo, il prodotto sarà attivato.

![Desktop View](https://user-images.githubusercontent.com/13579382/126905366-98d5e970-9ab5-4d44-bd36-9262728cff34.png)

![Desktop View](https://user-images.githubusercontent.com/13579382/126905367-ae20a076-380d-492d-a8aa-d51a6c3a81f1.png)





*Reversed with ❤ By\
Alessandro Sgreccia,\
Cyber Security Enthusiast*
