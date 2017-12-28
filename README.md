# Ordini Bloccati Macro Excel VBA

Questo progetto descrive l'automazione della reportistica degli ordini bloccati nell'ambito delle attività di recupero credito di un'azienda del settore farmaceutico

``` vb

Private Sub Orders_Blocked()
''----------------------------------------------------------
'' "Private" precede "Sub" per non fare apparire la macro nell'elenco delle macro 
'' (accessibile con la combinazione di Tasti ALT-F8 nell'interfaccia di Excel)
'' Macro VBA per Elaborazione ordini bloccati causa assenza pagamenti
'' Prima fase: preparare il foglio Excel all'inserimento dei commenti
'' Notare che la struttura dei fogli deve rimanere la stessa!
''----------------------------------------------------------
'' NB "Apostrofo" corrisponde a Commenti quindi Visual Basic ignora
''     "Spazio + Trattino basso _ " serve in un'istruzione per continuare su
''     una nuova riga quando l'istruzione supera lo schermo
''

''Aprire il file  OB_C.xlsx prima di avviare la Macro!!!

'' Dichiarazione della variabile "infos" a cui verrà assegnata la scelta dell'utente
'' se eseguire o no la Macro: Utile nel caso viene premuto inavertitamente

Dim infos As Variant  

'' "Msgbox" per fornire infos all'utente di ciò che succederà una volta che avra premuto su ok
'' L'uso del "&" serve per concatenare le stringhe

infos = MsgBox("Elaborazione file Ordini Bloccati..." & vbNewLine & vbNewLine _
    & "Salvare Prima!!! ==>> CTRL-Z non funziona!" & vbNewLine & vbNewLine _
    & "Accertarsi salvataggio File necessari nel seguente percorso." _
    & vbCrLf & "T:\CONTABILITA'\RECUPERO CREDITI\macraris_kl\MacrAris\Orders_Blocked_Macro" _
    & vbCrLf & vbCrLf & "Pregasi NON ALTERARE il nome dei File" _
    & vbNewLine & vbNewLine & "" & vbNewLine & "Qui per sbaglio -->  Click 'NO'", _
      vbYesNo + vbInformation + vbDefaultButton2, "Macr@ris Ordini Bloccati")
                    
'' La seguente condizione valuta se l'utente ha cliccato su "NO" 
'' in caso affermativo l'esecuzione della macro si interrompe son "Exit Sub"
'' "Exit Sub" corrisponde a esci dalla Routine

    If infos = vbNo Then 
        Exit Sub '' Non esegue la macro in quanto l'utente ha lanciato la macro per errore
    End If '' Fine esecuzione Macro
```

### Esecuzione routine esterna creazione Cartelle anno successivo

```vb
'' Macro nidificata in un'altra: richiama un'altra macro (codici vba in indice)
'' Utile a fine Anno Dicembre: crea automaticamente la cartella relativa
'' al nuovo anno di riferimento in cui salvare i file elaborati

creazioneCartelleYr1_ordersblock

```
### Impostazione delle diverse cartelle Excel

```vb
'' Dichiara una variabile di tipo cartella
Dim wOb_C As Workbook  
''Assegna ed inizializza la cartella attiva alla variabile wOb_C
Set wOb_C = ActiveWorkbook 

''controlla che il file attivo è effettivamente quello di interesse e se negativo
'' visualizza un messaggio informativo e poi interruzione. E' utile includere la
'' la gestione di errori prevedibili quando si sviluppa una Macro.

If wOb_C.Name <> "OB_C.xlsx" Then
 MsgBox "File Excel NON Corretto!" _
        & vbCrLf & vbCrLf & "verifica che file attivo sia OB_C.xlsx" _
        & vbCrLf & vbCrLf & "Interuzione Macro senza Alcuna Conseguenza!", _
        vbCritical, "Macr@ris messaggio di errore"
 Exit Sub ''Fine esecuzione Macro
End If

'' Dichiarazione variabili per rilevare la durata di elaborazione della Macro
Dim triggerChrono As Date, endtriggerChrono As Date, Interval As Date, strOutput As String
''Web Fonte dati
''http://msdn.microsoft.com/en-us/library/office/ff197413(v=office.15).aspx

'' alla Variabile triggerChrono viene assegnata l'ora al momento dell'esecuzione
'' dell'istruzione che segue
triggerChrono = Now

''Messaggio all'utente dell'esecuzione in corso della Macro nella barra di stato
Application.StatusBar = "Elaborazione Dati in Corso... Goditi un pò di Relax!"

'' Movimenti dello schermo possono rallentare l'esecuzione delle macro
''Il valore "False" disattiva i movimenti dello schermo
Application.ScreenUpdating = False

'' Ad ogni errore riscontrato durante l'esecuzione della macro la sua gestione è rinviata a
'' "ErrorHandler" dove ci sono delle istruzioni che catturano l'errore e l'utente è informato
'' Tramite commando "Msgbox" sul tipo di errore e la sua descrizione e cosa fare

  On Error GoTo ErrorHandler

''auto adattamento larghezza colonna A
  Columns("A:A").EntireColumn.AutoFit
  [b1].Value = "Valore Ordini"    '' nuovo valore della cella B1
  Range("B1").Font.Bold = True ''Applica il grassetto alla cella B1
  Columns("B:B").EntireColumn.AutoFit '' auto adattamento larghezza colonna B
    
[H1].Value = "Rag. Sociale"  '' inserisce il testo tra virgolette nella cella H1
[T1].Value = "Rif. Ord. Cliente"

''Applica larghezza fissa con relative misure alle colonne indicate
    Columns("C:C").ColumnWidth = 1.5
    Columns("D:D").ColumnWidth = 4.33
    Columns("E:E").ColumnWidth = 2.17
    Columns("F:F").ColumnWidth = 2.5

''inserisce una colonna in posizione G    
  [G:G].Insert Shift:=xlToRight, copyorigin:=xlFormatFromLeftOrAbove
  Columns("G:G").ColumnWidth = 55

'' con la struttura "With...End With" attribuisce una serie di caratteristiche alla cella G1       
   With Range("G1")  
        .Value = "Azioni / Commenti"
        .Font.Bold = True
        .Font.Name = "Arial"
        .HorizontalAlignment = xlCenter
        .VerticalAlignment = xlCenter
        .Font.Size = 12
    End With
'' applica il formato date indicato alle colonne K e L    
     Columns("K:L").NumberFormat = "dd-mmm-yy"  
    
    [U:U].Cut '' Rimuove la colonna U
'' inserisce la colonna U prima della colonna N spostandola verso destra
    [N:N].Insert Shift:=xlToRight 

'' Elimina le colonne O a Z spostando le altre verso sinistra
    Columns("O:Z").Delete Shift:=xlToLeft 
''Applica adatta colonna alle colonne da H a N    
    [H:N].EntireColumn.AutoFit 
    
 Dim shOb_C As Worksheet '' Dichiarazione di una variabile come oggetto foglio

'' Assegna il foglio 1 alla variabile foglio dichiarata alla riga precedente
    Set shOb_C = wOb_C.Sheets(1)
'' Rinomina il foglio rappresentato da shOb_C    
    shOb_C.Name = "Ordini Bloccati-clienti arancio"
``` 

### Copiare Fogli da una Cartella a un'altra in Excel VBA

```vb
'' Aggiunta del foglio Finservice
'' "Ciclo For ... Each" per eseguire un stesso gruppo di commando su tutti i fogli
'' nella cartella di lavoro

Dim CrSh As Worksheet '' Dichiarazione di una variabile oggetto Foglio
'' Disattiva tutti messaggi d'avvertimento quando si elimina un oggetto   
    Application.DisplayAlerts = False

'' ciclo per esecuzione su tutti i fogli       
    For Each CrSh In Worksheets 
    ''Condizione IF da controllare per ogni ciclo
        If CrSh.Name <> shOb_C.Name Then 
''Elimina foglio se nome non uguale a shOb_C.Name = "Ordini Bloccati-clienti arancio"
        CrSh.Delete 
        End If '' fine esecuzione condizione
'' "Next" riporta il codice di esecuzione a "For" per eseguire 
'' gruppo di codici IF...End IF sul foglio successivo        
    Next

'' Riattiva i messaggi di avvertimento a seguito eliminazione oggetto        
    Application.DisplayAlerts = True 
    
'' Matrice Aris Cerca verticale su file compensi collectors
'' La struttura matriciale è molto più veloce che lavorare sui
'' file Excel

 Dim Cy As String
 '' Assegna alla variabile "Cy" l'anno corrente
    Cy = Format(Now, "YYYY")
    
''Dichiarazione di una variabile oggetto Cartella    
Dim wCompensiColl As Workbook
''apertura di una cartella di lavoro e sua assegnazione alla variabile
'' tipo cartella appena creata
   Set wCompensiColl = Workbooks.Open(Filename:= _
    "T:\CONTABILITA'\RECUPERO CREDITI\Credit_Collectors\Contratti_Credit_Collectors\" _
      & Cy & "\tabella_compensi_collectors.xlsx")

'' Dichiarazione di una variabile di tipo foglio
Dim ShCompensiC As Worksheet
'' Assegnazione del foglio "BBMI_PRIV_COLLECTORS" alla variabile di tipo foglio
   Set ShCompensiC = wCompensiColl.Sheets("BBMI_Priv_Collectors") 

'' attiva il foglio denominato "Ordini Bloccati-clienti arancio" tramite la variabile shOb_C
shOb_C.Activate

''In questo blocco si difinisce la matrice virtuale in cui copiare i dati 
'' per eseguire  la ricerca verticale. la creazione di una matrice virtuale
'' rende parecchio più veloce l'elaborazione dei dati

Dim vAIndexCliente As Variant, vaNameColl As Variant
Dim avlookup As Variant
Dim avResult() As Variant '' Notare il "()" che dichiara una matrice

''Uso di "With....End With" come scorciatioa per assegnare i valori
'' alla matrice nominata VaNameColl

With ShCompensiC 
  vaNameColl = .Range(.Cells(.Range("A1").SpecialCells(xlCellTypeLastCell).Row, "A"), "I2")
End With

Application.DisplayAlerts = False
    wCompensiColl.Close ''Chiusura della cartella
Application.DisplayAlerts = True

''Questa alternativa eliminata perche rendeva il file molto pesante e voluminoso
    '' avlookup = Range(Cells(Rows.Count, "H").End(xlUp), "H2")
'' Selezione dinamica dell'intervallo di dati H2 e
avlookup = Range(Cells(Range("H1").SpecialCells(xlCellTypeLastCell).Row - 1, "H"), "H2") 

''l'ultima cella dell'intervallo che contiene i dati.
''notare l'uso "Vai a formato speciale Ultima cella.
''identifica la riga e meno 1 per avere il numero di riga che contiene
''l'ultimo dato
''Assegna poi l'intervallo di dati selezionati alla matrice "avLookup"
 
 '' Con "ReDim" crea un intervallo di dati in matrice di n righe, 1 colonna della stessa
 '' dimensione della matrice "AvLookup"
 ReDim avResult(1 To UBound(avlookup, 1), 1 To 1) 

''ciclo di ripetizione con limite di esecuzione n esima riga della matrice avlookup
For i = 1 To UBound(avlookup, 1)

''ignorare eventuali errori generati nell'esecuzione della macro specie quando il risultato della
'' formula CERCA.VERT restituisce un #N/D
    On Error Resume Next  
'' CERCA.VERT del dato in riga i nell'intervallo di dati VanameColl    
avResult(i, 1) = WorksheetFunction.VLookup(avlookup(i, 1) * 1, vaNameColl, 9, 0)

'' se errore di tipo 1004 allora risultato CERCA.VERT = #N/D quindi sostituisci col Nome
    If Err.Number = 1004 Then avResult(i, 1) = "Mombrini"  

Next i

'' Ripristina la gestione degli errori generici definita per l'insieme della Macro
    On Error GoTo -1
    On Error GoTo ErrorHandler
    
 '' copia i risultati della ricerca verticale nell'intervallo limite inferiore cella O2
 '' e limite superiore n righe della matrice Avlookup                
    [O2].Resize(UBound(avlookup, 1), 1).Value = avResult 
    
With Range("N1")
    .Copy [O1]
        With .Offset(0, 1)
            .ClearContents
            .Value = "Collectors"
        End With
    
End With
  
''Termine CERCA.VERT (VLOOKUP) con struttura a matrice su
'' Cartella di lavoro compensi collectors
```
### ...Prosegue la Macro con Ricerca Verticale ("VLOOKUP") su 2 cartelle

```vb

'' Applica Cerca verticale su file bloccati settimana precedente
    
 Dim wOb_P As Workbook ''Dichiarazione di variabile oggetto di tipo cartella
 
 '' Apre un cartella specifica di nome "OB_P.xlsx"
 ''Assegna il tutto alla variabile inizializzata cartella "wOb_P
   Set wOb_P = Workbooks.Open(Filename:= _
    "T:\CONTABILITA'\RECUPERO CREDITI\macraris_kl\MacrAris\Orders_Blocked_Macro\OB_P.xlsx") 

''Dichiarazione di variabile oggetto di tipo foglio
Dim shOb_P As Worksheet

''Assegna il foglio denominato "Ordini Bloccati-clienti arancio" alla variabile "shOb_P"
   Set shOb_P = wOb_P.Sheets("Ordini Bloccati-clienti arancio")
    Columns("H:H").Cut '' taglia la colonna
'' inserisce la colonna h tagliata e sposta la colonna G verso destra
    Columns("G:G").Insert Shift:=xlToRight

 ''Seleziona l'intervallo di dati utili nella colonna G2
    With shOb_P
        .Range(.Cells(.Range("G1").SpecialCells(xlCellTypeLastCell).Row, "G"), "G2").Select
    End With
 '' Applica la funzionalità testo in colonne per trasformare i dati in formato testo
 '' Utile per la funzione "CERCA.VERT" in quanto i dati che servono per la ricerca
 '' devono essere dello stesso tipo
       Selection.TextToColumns Destination:=Range("G2"), DataType:=xlFixedWidth, _
            FieldInfo:=Array(0, 2), TrailingMinusNumbers:=True                             

''Attiva il foglio assegnato alla variabile shOb_C       
shOb_C.Activate

'' Copia fogli dalla cartella (file) settimana precedente dentro la corrente cartella
'' Utilizza la struttura a matrice per copiare + fogli 
    wOb_P.Sheets(Array("Finservice_Affidati", "clienti a contenz o pro concors", _
            "privati payer")).Copy After:=shOb_C
 
 ''attiva il foglio ; vedi il foglio a cui e' stato assegnato la variabile
 shOb_C.Activate
''Nel file settimana precedente assegna l'intervallo di dati
'' tra nelle colonne G a H alla matrice vaNameColl
With shOb_P
    vaNameColl = _
    .Range(.Cells(.Range("G1").SpecialCells(xlCellTypeLastCell).Row, "G"), "H2")
End With

''disattiva le notitiche di Excel
    Application.DisplayAlerts = False
''chiude la cartella di lavoro settimana precedente senza salvare    
        wOb_P.Close
''Riattiva le notifiche di Excel
    Application.DisplayAlerts = True

For i = 1 To UBound(avlookup, 1) '' Ciclo di ripetizione
    On Error Resume Next ''Ignora eventuali errori
'' Cerca verticale sulla matrice di dati per copiare i commenti presenti
'' nel file della settimana precedente. Notare che prima di chiudere il file con
'' variabile "wOb_P" tali valori sono stati assegnati alla matrice vaNameColl.
'' Ove possibile preferire operazioni sulle matrici al posto delle operazioni 
'' sulle celle di Excel in quanto la velocita di elaborazione e' di 10+
'' Quando riscontra un errore di tipo 1004 con CVErr(xlErrNA)  assegna il valore #N/D
        avResult(i, 1) = WorksheetFunction.VLookup(avlookup(i, 1), vaNameColl, 2, 0)
            If Err.Number = 1004 Then avResult(i, 1) = CVErr(xlErrNA)
Next i

''Ripristina la gestione di errori generici
On Error GoTo 0
On Error GoTo ErrorHandler

'' Restituisce i risultati della ricerca verticale dalla cella G2 in poi.
'' I risultati sono presi dalla matrice avResult
    [G2].Resize(UBound(avlookup, 1), 1).Value = avResult
       
''### Conclude la parte di uso di funzione "CERCA.VERT" su file 
'' Ordini bloccati della settimana precedente.
'' Notare che mentre in Excel abbiamo la traduzione della funzione
'' CERCA.VERT, IN VBA tutti i commandi sono in INGLESE QUINDI
'' PER LA RICERCA VERTICALE SI USA "VLOOKUP"
```

### ...Prosegue con un interessante esempio su Cicli For...Next e Unisci Celle

```vb
''blocca riquadri da posizione cella J2
    ActiveWindow.ScrollColumn = 1
    Range("J2").Select
    ActiveWindow.FreezePanes = True

''Sub LoopTroughOrdersBlocked() ' sviluppato iniazialmente a parte!
'' Per il buon funzionamento di questa macro, la colonna numero 8 deve essere ordinata in modo
'' crescente. La macro esamina e seleziona i duplicati nella colonna 8 e quindi nella colonna 
'' corrispondente alla selezione fa un UNISCI CELLE quindi lo scopo è unire e centrare tutte le
'' celle in corrispondenza di più posizioni dello stesso cliente nella colonna "Azioni/Commenti"

Cells(2, 8).Activate  ''Seleziona la cella riga 2 colonna 8

Dim x As Integer, y As Integer ''Definizione di variabile di traccia riga

x = ActiveCell.Row   ''L'attuale riga viene assegnata a x
y = x + 1              '' la riga x+1 viene assegnata a y

'' Loop A esegue il blocco di codici ripetutamente finche la cella non sara vuota
'' e a quel punto si fermera' il Do While mentre il loop B agisce sui duplicati
''' di codici clienti per i quali poi applicare "Unisci Celle"
   
Do While Cells(x, 8).Value <> ""  
    If Cells(x, 8).Value <> Cells(y, 8).Value Then
        Cells(y, 8).Select
    Else
 '' Loop B    
Do While Cells(x, 8).Value = Cells(y, 8).Value
 
        If Cells(x, 8).Value = Cells(y, 8).Value Then
            Range(Selection, Selection.Offset(1, 0)).Select
        End If
        
        x = x + 1
        y = x + 1
''Fine Loop B 
        Loop
'' Con "Scarto" seleziona le celle colonna adiacente sinistra
          Selection.Offset(0, -1).Select

'' Riuso di routine in quanto serve anche per altre macro
'' Applica alle celle selezionate "Unisci Celle"
        Application.Run "PERSONAL.XLSB!Merge_Cells"

'' Sposta la selezione una riga sotto e una colonna a destra
    Selection.Offset(1, 1).Select
End If

 x = x + 1
        y = x + 1

''Fine Loop A
Loop
''#---------------- Termina la parte sui cicli ripetuti e Unisci centra Celle
```

### Aggiunta di funzione "SOMMA" e "SCARTO" su intervalli dinamici
#### Nel senso che possono essere 10 righe come 100

```vb

    Cells.Select '' Seleziona tutte le celle
 '' Alla selezione tutte celle attribuisce altezza righe 15
    Selection.RowHeight = 15

''@# Questa sezione  somma il valore di tutti gli ordini inserendo una formula

Dim PrimaCella, lastsumCella As String ''Dichiarazione di due variabili di tipo stringa

'' Assegnazione della stringa B2 alla variabile prima cella; 
'' servirà per indirizzo di cella nella formula
PrimaCella = "B2"

'' Con la funzione "OFFSET" e "ADDRESS" si rileva l'indirizzo
'' dell'ultima cella contenente un valore (agisce in modo dinamico)
lastsumCella = Range("B2").End(xlDown).Offset(-1, 0).Address(rowrelative, columnrelative)

'' Inserisce la formula somma nell'ultima cella di valore
Range("B2").End(xlDown).Value = "=sum(" & PrimaCella & ":" & lastsumCella & ")"

'' applica formato con separatore migliaia a intervallo dati colonna B
    Range(Cells(Range("B1").SpecialCells(xlCellTypeLastCell). _
        Row - 1, "B"), "B2").NumberFormat = "#,##0.00_);(#,##0.00)"
'' Applica formato numero con Euro
    Cells(Range("B1").SpecialCells(xlCellTypeLastCell).Row, "B").NumberFormat = "€ #,##0"

    
Cells(1, 1).Select '' Seleziona la cella A1

Dim stAttachment As String '' Dichiarazione Variabile stringa
Dim StPath As String

''Assegnato il percorso ove salvare il file a Stpath
    StPath = "T:\CONTABILITA'\RECUPERO CREDITI\file ordini bloccati\BBMI_" _
                            & Cy & "_Ordini_Bloccati\"

''Dichiarazione della variabile per il nome file                            
  Dim StFileName As String
''assegna nome file con stringa piu' funzione FORMAT per la data
    StFileName = "Ordini_Bloccati_" & Format(Date, "DD-MM-YYYY")
''percorso completo di salvataggio attribuito a StAttachment    
    stAttachment = StPath & StFileName & ".xlsx"
                  
    Application.DisplayAlerts = False  '' disattiva tutte le notifiche di avvertimento
        With ActiveWorkbook  ''con la struttura with salva il file nel percorso definito prima
            .SaveAs stAttachment ''", FileFormat:=xlOpenXMLWorkbook" argomento facoltativo
       End With
    Application.DisplayAlerts = True  '' Attiva il messaggio di avvertimento

'''Riattiva i movimenti dello schermo quando si esegue la macro
    Application.ScreenUpdating = True
 
 ''...preso dalla mia Macro tracking time
  endtriggerChrono = Now   ''Rileva l'orario di fine Esecuzione
 'Calcola la durata dell'esecuzione
Interval = endtriggerChrono - triggerChrono

 '' Formato della durata in minuti e secondi
     strOutput = Int(CSng(Interval * 24 * 60)) & " Minutes :" & Format(Interval, "ss") _
        & " Seconds"
    ''Debug.Print strOutput "Ha servito solo per test
    '' il valore viene visualizzato nella finestra immediata

'' Messagio finale di fine elaborazione e durata di esecuzione
'' Notare l'uso variabile strOutput
        MsgBox " Durata Elaborazione Bloccati" & vbCrLf & vbCrLf _
            & strOutput, vbOKOnly + vbInformation, "Macr@ris Tracking Time"
 
 '' riattiva le impostazioni predefinite della barra di stato
 Application.StatusBar = ""
 Exit Sub '' invoca Fine esecuzione macro se errori non riscontrati
 ''#-------------

'' Gestione di errore con rilevamento tipo di errore e descrizione
ErrorHandler:
        MsgBox "Interruzione Macro Causa Errore in Orders_Blocked" & vbNewLine _
            & "Contattare Macr@ris" & vbNewLine _
            & vbCrLf & "Error number:  # " & Err.Number & vbNewLine _
            & "Description:==> " & Err.Description, vbCritical, "Macr@ris \Error Macro"
 
Application.ScreenUpdating = True  ''riattiva i movimenti dello schermo
Application.DisplayAlerts = True  '' riattiva tutti i messagi di avvertimento
Application.StatusBar = ""        '' riattiva le impostazioni predefinite della barra di stato

End Sub ''Istruzione di fine Esecuzione

```
## Creazione Dinamica cartelle

_Questa macro richiamata dalla macro..._

> verifica nel mese di dicembre che la cartella ove salvare il file di elaborazione
> sia disponibile e in caso contrario procede alla creazione della cartella
> e avvisa l'utente. Controllo effettuato dal 15 dicembre di ogni anno


Il controllo è esgeguito su 2 cartelle
1. "T:\CONTABILITA'\RECUPERO CREDITI\file ordini bloccati\
2. "T:\CONTABILITA'\RECUPERO CREDITI\Credit_Collectors\Contratti_Credit_Collectors\"

> Inoltre se il file "tabella_compensi_collectors.xlsx" non è presente viene copiato quello dell'anno
> Precedente con avviso all'utente di aggiornare il file appena possibile

```vb
'' Sub crea la routine 
	Sub creazioneCartelleYr1_ordersblock()

'' Gestione errori
	On Error GoTo ErrorHandler

'' Verifica che il mese sia dicembre
	If Format(Now, "mm") = "12" Then

'' Dichiarazione di variabili per impostazione dell'intervallo temporale di controllo

	Dim intDate As Date, checkDateInf As String, checkDatesup As String, yrInterval As String

'' Data del giorno assegnata a variabile intDate
	intDate = Date
	
''Formatto anno successivo in YYYY assegnato a yrInterval
	yrInterval = Format(WorksheetFunction.EoMonth(Now(), 1), "YYYY")
	
'' Limite inferiore intervallo di controllo data (variabile di tipo stringa)
	checkDateInf = "12/14/" & Format(intDate, "YYYY")
	
''Limite superiore intervallo di controllo data (variabile di tipo stringa)
	checkDatesup = "12/31/" & Format(intDate, "YYYY")
	
''Condizione IF di verifica da data del giorno si trova nell'intervallo di controllo
''NB- "DateValue" converte la variabile stringa in data per rendere possibile il 
''controllo
 If intDate > DateValue(checkDateInf) And intDate < DateValue(checkDatesup) Then
'' Se siamo tra il 15 e 31 dicembre dell'anno allora

'' Dichiarazione di un oggetto
	Dim objFso As Object
'' Initializzazione dell'oggetto creato
        Set objFso = CreateObject("Scripting.FileSystemObject")
		
        Dim scheckPath As String, scheckFolder As String
 '' Attribuzione percorso file alle variabili       
        scheckPath = "T:\CONTABILITA'\RECUPERO CREDITI\file ordini bloccati\"

'' Verifica esistenza cartella anno successivo
'' Notare l'uso della variabile yrInterval pare l'anno di riferimento della cartella
        scheckFolder = "BBMI_" & yrInterval & "_Ordini_Bloccati"
 
 '' Se la cartella esiste già allora niente quindi prossima istruzione = End if  
   If objFso.FolderExists(scheckPath & scheckFolder) Then
            ''do Nothing
             Else ''' altrimenti crea detta cartella
                 objFso.CreateFolder (scheckPath & scheckFolder)		
 '' con MsgBox informa l'utente che tale cartella e' stata creata 
                MsgBox "AVVISO Creazione Cartella Prox Anno!!!" & vbCrLf & vbCrLf & _
                "è stata creata la cartella " & scheckFolder & vbCrLf & _
                "Nel seguente percorso:" & vbCrLf & scheckPath, vbInformation, _
		"Macr@ris Cartella Automatica Ordini"
 '' Fine della condizione IF               
    End If
 ```
 
 ### ... prosegue verifica file tabella_compensi_collectors.xlsx 
 > Se assente nella cartella anno successivo viene copiata la cartella
 > anno corrente nel percorso creato anno successivo
 > _Ricordare nel msgbox va aggiornato sulla base nuova contrattazione_
 
```vb
'' Nuova assegnazione di variabile  
  scheckPath = "T:\CONTABILITA'\RECUPERO CREDITI\Credit_Collectors\Contratti_Credit_Collectors\"
        scheckFolder = yrInterval
		
'' Condizione if di verifica se cartella esiste
    If objFso.FolderExists(scheckPath & scheckFolder) Then
             
              Dim cryInt As String
                            cryInt = Format(Now, "YYYY") & "\tabella_compensi_collectors.xlsx"
							
 '' Condizione if di verifica se file esiste                           
	If objFso.FileExists(scheckPath & scheckFolder & _
			"\tabella_compensi_collectors.xlsx") = False Then
                           
'' se test condizione falsa allora copia file corrente dentro la cartella anno successivo
	FileCopy scheckPath & cryInt, scheckPath & scheckFolder & "\tabella_compensi_collectors.xlsx"

'' MsgBox a utente per informazione di copia di file avvenuta e quindi di aggiornamento quanto prima                       

	MsgBox "AVVISO di Verifica File!!!" & vbCrLf & vbCrLf _
	& "Nella cartella " & scheckFolder & vbCrLf _
	& "Percorso:" & vbCrLf & scheckPath & vbcrLf & vbCrLf _
	& "Ricordarsi di aggiornare il file 'tabella_compensi_collectors.xlsx'" & vbCrLf _
	& "Update in base alle nuove allocazioni contrattuali clienti", vbInformation, _
	"Macr@ris Cartella Automatica Orders"
'' Altrimenti se il file esiste già informare semplicemente l'utente di aggiornare tale file                    
     Else
	MsgBox "AVVISO di Verifica File!!!" & vbCrLf & vbCrLf _
	& "Nella cartella " & scheckFolder & vbCrLf _
	& "Percorso:" & vbCrLf & scheckPath & vbCrLf _
	& vbCrLf & "Ricordarsi di aggiornare il file 'tabella_compensi_collectors.xlsx'" _
	& vbCrLf & "Update in base alle nuove allocazioni contrattuali collectors", _
	vbInformation, "Macr@ris Cartella Automatica Orders"
                                        
      End If
'' Se la cartella non esiste: creazione della cartella e copia al suo interno del file
'' del corrente anno 
    Else
	objFso.CreateFolder (scheckPath & scheckFolder)
	FileCopy scheckPath & cryInt, scheckPath & scheckFolder & "\tabella_compensi_collectors.xlsx"

	MsgBox "AVVISO Creazione Cartella Prox Anno!!!" & vbCrLf & vbCrLf _
	& "è stata creata la cartella " & scheckFolder & vbCrLf _
	& "Nel seguente percorso:" & vbCrLf & scheckPath & vbCrLf _
	& vbCrLf & "Ricordarsi di aggiornare il file 'tabella_compensi_collectors.xlsx'" & _
	"salvato nella nuova cartella creata", vbInformation, "Macr@ris Cartella Automatica Orders"
                  
    End If
 End If

End IF

'' Termina qui se errore non riscontrato

Exit Sub

'' in Caso di errore va qui
ErrorHandler:
MsgBox "Interruzione Macro causa errore in 'CreazioneCartelleYr1_ordersblock'" & "   " _
& vbNewLine & vbCrLf & "Descrizione: - " & Error(Err) & vbCrLf _
& "Numero Errore #-" & Err, vbOKOnly + vbExclamation, "Macr@ris Msg Errore"

'' End Sub per concludere la routine
End Sub
```
