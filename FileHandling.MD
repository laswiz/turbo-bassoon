### [HOME](https://laswiz.github.io/turbo-bassoon/index.html)

### Let's Handle Some Files!
 Windows doesn't do a lot of things well, but it DOES handle files pretty nicely. So when an opportunity to update and simplify file handling presented itself, I took it.

To review, before I started, everything was done in the control. This includes file handling, importing and exporting data and programs, even handling the touch screen 
is in the machine control. This works great if nothing ever changes, and you place strict requirements on customers and how they use your machine. When you force all 
programs to be imported from a USB stick, but can only recognize the first one, that’s a problem. If Windows decides that on Monday, the USB stick will be drive letter H, 
but on Tuesday, it is L, that’s a problem. Putting the onus on the PLC is asking for trouble. My view is Windows is very good at managing files, so why not let it? That 
is what I did. I also did it with Powershell, so that any modifications can be done without software updates (if the changes required are solely to the Powershell side). 
But I didn’t stop with file handling. Other mundane tasks I also put into automated powershell scripts, and I have even added functionality. This part of my portfolio 
showcases me directly, my approach to problems, my methodology and thinking as to problem solving, and my technical prowess. It is exceedingly difficult to show the before 
code, because it belongs in one gigantic, monolithic module that is nearly 10,000 lines long, and has much more in it than just file handling. Add to that the pages of code 
for the HMI, the text and error messages, and it would take me hours to explain everything when a few pictures would do. So without further ado, here is how the file 
handling worked before I started with this company:

![Picture 1](https://laswiz.github.io/turbo-bassoon/assets/Picture1.png)


File selection. Note there is only one accessible directory called “C”, but this is actually on another drive!
Import and Export shared similar screens (but were different):

![Picture 2](https://laswiz.github.io/turbo-bassoon/assets/Picture2.png)

Saving a file was another process entirely:

![Picture 3](https://laswiz.github.io/turbo-bassoon/assets/Picture3.png)

I think you get the idea. Importing and exporting were limited to one drive letter, and if the USB stick was not on that letter, it just failed. Worst of all, the only 
way to fix it was to reboot the control! On to happier things. What I did to fix it, as I already mentioned, was to incorporate Powershell. Here is the file open function:

![Picture 4](https://laswiz.github.io/turbo-bassoon/assets/Picture4.png)

You can see in my example I have 3 drives available, one is a USB, one is local, and one is a network drive. The PLC doesn’t care where the file comes from, so long as 
it is formatted properly and moved to the transfer folder. I combined the import, export and file handling into one function. Why not? They are essentially doing the 
same thing!

![Picture 5](https://laswiz.github.io/turbo-bassoon/assets/Picture5.png)

This was modeled after programs available such as Total Commander. You can move files in either direction, delete them, move multiple files, rename files, etc. Just 
like Total Commander. I went further by also adding in a system import and export utility, a screenshot utility (which with the help of the PLC navigates through all 
the available pages in the HMI, creates a screenshot, makes a PDF and offers the user the opportunity to save that file somewhere). I help the user set the time and 
date (this control manages 76 different time zones!), and I also control EWF/UWF to protect the flash drive from too many writes. All this came at a cost of almost 
1500 lines of code saved, and an HMI that is nearly 5MB smaller because of the pages removed. Add to that the PLC is more responsive, more robust and more user friendly, 
and you can see why this was a no-brainer. The only downside is that because I use Windows Forms, the first time it loads, it takes about 3 seconds for the form to appear, 
which is slower than before. I am looking at ways that I can mitigate this, but as of yet, there is no simple way; it’s a Microsoft thing.

The last thing I want to do is to provide some of the code that I used for these functions. There is nothing proprietary here, it's all stuff I researched and implemented
to work on my machine control. I provide it as context for how I code. This first part is just setting up all the form controls, please note how the name of my controls
is easy to understand what they are (it's not button 1 for example):]. It also sets up the system and a global path used throughout the module:

```
[void][System.Reflection.Assembly]::LoadWithPartialName("System.Drawing") 
[void][System.Reflection.Assembly]::LoadWithPartialName("System.Windows.Forms")
[System.Windows.Forms.Application]::EnableVisualStyles()
#global info
$ErrorActionPreference = "Stop"

#define path array for all drives... The first element is always the local C drive (will be different on the machine)
$global:drivePathArr = New-Object System.Collections.ArrayList
$global:currentPath = ''

#define form information and other global information
$OpenFile = New-Object System.Windows.Forms.Form
$OpenFile.ClientSize = '520,615' #set size
$OpenFile.Text = 'Open Coiler Program'
$OpenFile.TopMost = $true
$OpenFile.StartPosition = "CenterScreen"
$OpenFile.FormBorderStyle = 'Fixed3d'
$OpenFile.ControlBox = $false #disable the ability to close the form


#create a label with instructions on the usage (create a string variable so you can format the text appropriately
$tempStr = -join
(
    'Available drives are listed in the drop down box. Select the drive; the folder contents are shown in the box below.',
    ' Double click or press OK to load program. Cancel will abort the process and return to the HMI. If a drive contains',
    ' a folder, the name is shown without a date. Double click to navigate into it. Use the < to navigate up one folder.',
    ' Choose a new drive to reset the file system.'
)
$lblTop = New-Object System.Windows.Forms.Label
$lblTop.Text = $tempStr
$lblTop.Width = 500
$lblTop.Height = 75
$lblTop.Font = 'Times New Roman, 10'
$lblTop.Location = New-Object System.Drawing.Point(10, 15)

$lstFiles = New-Object System.Windows.Forms.ListView
$lstFiles.width = 500
$lstFiles.Height = 410
$lstFiles.Font = 'Arial Black, 12'
$lstFiles.Location = New-Object System.Drawing.point(10, 160)
$lstFiles.View = 'Details'
$lstFiles.FullRowSelect = $true
$lstFiles.GridLines = $true 
$lstFiles.MultiSelect = $false
[void]$lstFiles.Columns.Add("Filename", 275);
[void]$lstFiles.Columns.Add("Date", 200);

$lblFilter = New-Object System.Windows.Forms.Label
$lblFilter.Text = "Filter Results:"
$lblFilter.Width = 140
$lblFilter.Height = 75
$lblFilter.Font = 'Arial Black, 12'
$lblFilter.Location = New-Object System.Drawing.Point(10, 125)

$txtFilter = New-Object System.Windows.Forms.TextBox
$txtFilter.Location = New-Object System.Drawing.Point(180,125)
$txtFilter.AutoSize = 'False'
$txtFilter.Width = 280
$txtFilter.Height = 150
$txtFilter.Font = 'Arial Black, 12'
$txtFilter.BorderStyle = "Fixed3d"
$txtFilter.Text = ""

#create a combo box to house all the possible drives
$cmbDrives = New-Object System.Windows.Forms.ComboBox
$cmbDrives.width = 450
$cmbDrives.Height = 35
$cmbDrives.Font = 'Arial Black, 12'
$cmbDrives.Location = New-Object System.Drawing.Point(10,90)


#create the cancel button
$btnCancel = New-Object System.Windows.Forms.Button
$btnCancel.Text = 'Cancel'
$btnCancel.Width = 120
$btnCancel.Height = 35
$btnCancel.Font = 'Times New Roman, 14'
$btnCancel.Location = New-Object System.Drawing.Point(380, 578)

#create the OK button
$btnOK = New-Object System.Windows.Forms.Button
$btnOK.Text = 'OK'
$btnOK.Width = 120
$btnOK.Height = 35
$btnOK.Font = 'Times New Roman, 14'
$btnOK.Location = New-Object System.Drawing.Point(235, 578)

$btnUpFolder = New-Object System.Windows.Forms.Button
$btnUpFolder.Text = '<'
$btnUpFolder.Width = 35
$btnUpFolder.Height = 35
$btnUpFolder.Font = 'Arial Black, 12'
$btnUpFolder.Anchor = 'Right, Bottom'
$btnUpFolder.Location = New-Object System.Drawing.Point(($cmbDrives.Left + $cmbDrives.Width + 15), ($cmbDrives.Top - 1))
```

Next up is the form controls. We have to handle some events here when the user performs some functions. I put them together so they are easy to follow:

```
$btnOK.Add_Click( {
    #see if we have a file or a folder
    if ($lstFiles.SelectedItems -like '*.xml*') { 
        loadFile
    }
    else {

        # if it is not a file, it must be a folder, so attempt to navigate into it!
        #build the path to go to

        #update the global path
        $global:currentPath = $global:currentPath + "\" + $lstFiles.SelectedItems[0].Text
        fillListBox($global:currentPath)
    }

})

#click events for buttons
$btnCancel.Add_Click( {
    $OpenFile.Close()
    #eventual exit here
    [Environment]::Exit(99)
})

$btnUpFolder.Add_Click( {

    #based on the selection in the combo box, check the absolute path. If longer, then try and move up one folder
    if ($global:currentPath -ne $global:drivePathArr[$cmbDrives.SelectedIndex] ){
        #try and move up one folder
        #find the last "\", then replace the string with the previous folder
        $global:currentPath = $global:currentPath.Substring(0, $global:currentPath.LastIndexOf("\"))
        #call the listbox function
        fillListBox($global:currentPath)
    }
    #no else! Just stay here... we can't go any higher
})


#handle the combo box change event
$cmbDrives.Add_SelectedIndexChanged( {
    #fill the listview box with the new information
    #update the global variable
    $global:currentPath = $global:drivePathArr[$cmbDrives.SelectedIndex]
    fillListBox($global:drivePathArr[$cmbDrives.SelectedIndex])
    $txtFilter.Text = ""
})

#when you double click an entry, it could be to change folders or just load a file
$lstFiles.Add_DoubleClick( {

    #see if we have a file or a folder
    if ($lstFiles.SelectedItems -like '*.xml*') { 
        loadFile
    }
    else
    {
        #build the path to go to

        #update the global path
        $global:currentPath = $global:currentPath + "\" + $lstFiles.SelectedItems[0].Text
        fillListBox($global:currentPath)
     
    }


})

$txtFilter.Add_TextChanged( {
    #filter based on the entered text
    fillListBox($global:currentPath)
    
} )

```

Now we get to the heart of the matter. Here we are running a function to fill the listbox of the available files in the selected folder (which is a parameter):

```
function fillListBox ($pathtoSearch) {

    $lstFiles.items.Clear()
    $rootFolder = $pathToSearch
    $filterText = "*" + $txtFilter.Text + "*.xml"

    try
    {

        $subFolders = Get-ChildItem -Path $rootFolder -Directory 
        $subFolders | ForEach-Object {
            $row = New-Object System.Windows.Forms.ListViewItem($_.Name)
            [void]$lstFiles.items.Add($row)
        }

        $rootFolder = $rootFolder + "\*"
        #now get all the files
        #if the filter box is not empty, filter. Otherwise, show all
        if ( $txtFilter.TextLength -eq 0 ) {
            $subFiles = Get-ChildItem -Path $rootFolder -Include *.xml -File
        }
        else {
            $subFiles = Get-ChildItem -Path $rootFolder -Include $filterText -File
        }
        
        $subFiles | ForEach-Object {
            $row = New-Object System.Windows.Forms.ListViewItem($_.Name)
            [void]$row.SubItems.add($_.LastWriteTime.ToString())
            [void]$lstFiles.items.Add($row)       
        }
    }
    catch [System.Exception]
    {
        [System.Windows.Forms.MessageBox]::Show("Error! Drive Not Accessible! Choose a different drive", "Error Accessing Drive",[System.Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Exclamation)
    }
}
```

Next is a function to fill the combo boxes with available drives. Note, this includes some filtering functions to hide certain drives and devices:

```
#combobox clear and populate function
function fillComboBox() {
    $cmbDrives.Items.Clear()
    #reset the array
    $global:drivePathArr = New-Object System.Collections.ArrayList
    $global:drivePathArr.Add('F:\userdata') > $null 

    #add the 'C' drive... This is a fixed directory
    [void]$cmbDrives.items.add("(C:)   Local Drive")

    $USBDrives = Get-CimInstance Win32_Diskdrive -PipelineVariable disk | Where {$_.InterfaceType -eq 'USB' } | Get-CimAssociatedInstance -ResultClassName Win32_DiskPartition -PipelineVariable partition |   Get-CimAssociatedInstance -ResultClassName Win32_LogicalDisk 
    foreach ( $disk in $USBDrives ) {
        [void]$cmbDrives.Items.Add("(" + $disk.Name + ")   " + $disk.VolumeName)
        $global:drivePathArr.Add($disk.Name) > $null 
    }

    #get any network drives that are available as well
    $networkDrives = Get-WmiObject Win32_MappedLogicalDisk
    foreach ($drive in $networkDrives) {
        [void]$cmbDrives.Items.add("(" + $drive.Name + ")     " + $drive.ProviderName)   
        $global:drivePathArr.Add($drive.Name) > $null 
    }

    #select the first entry and then fill the listview
    $cmbDrives.SelectedIndex = 0
    #no need to auto fill, when we select index 0, the function will automatically fill!
    #however, update the global string we use
    $global:currentPath = $global:drivePathArr[$cmbDrives.SelectedIndex]
}
```

This function actually loads the file to the transfer folder for the PLC:

```
function loadFile() {
    #called from the double click event of the listview or the ok button
    
    #First step is to check if the file name has any specia characters.
    #these characters Windows does not allow and thus I do not check for them: / \ : * ? " < > |
    #in addition, these are not allowed in the control: ( ) ' ; (white space)
    #this is ugly, but do it with a huge if statement. For now, ignore a double point, since it is hard to check
    #use a temporary string so the variable name is shorter
    $str = $lstFiles.SelectedItems[0].Text
    if ( $str -match '\(' -or $str -match '\)' -or $str -match '\ ' -or $str -match '\;' -or $str -match "'" ) {

         [System.Windows.Forms.MessageBox]::Show("Error! Filename contains invalid character! Special characters are not allowed.", "Invalid Filename",[System.Windows.Forms.MessageBoxButtons]::OK, [System.Windows.Forms.MessageBoxIcon]::Exclamation)
    }
    else {
        #delete any file in the folder (for consistency)
        $ProgPath = "F:\txfr"

        #new file... delete any existing ones. Use recursive for any remaining folders, force if they are
        #not empty
        Remove-Item -path $progPath\* -Recurse -Force
        #now copy the file to the transfer folder
        
        $fileName = $global:currentPath + "\" + $lstFiles.SelectedItems[0].Text
        Copy-Item $fileName -Destination $progPath\
    
        #before we exit, we save the path data to an XML file
        #if a file of the same name exists, just blow it out (this way, we don't care if there was one there before)
        $storedPath = "F:\Scripts\SavedFileLocation.xml"
        Remove-Item $storedPath -ErrorAction Ignore

        #now save the data to an XML
        $xmlWriter = New-Object System.Xml.XmlTextWriter($storedPath, $Null)

        #setup formatting
        $xmlWriter.Formatting = 'Indented'
        $xmlWriter.Indentation = 4
        $xmlWriter.IndentChar = " "

        #writer the header
        $xmlWriter.WriteStartDocument()
        $xmlWriter.WriteProcessingInstruction("xml-stylesheet", "type='text/xsl' href='style.xsl'")

        #write the data
        $xmlWriter.WriteStartElement("Root")
        $xmlWriter.WriteStartElement("PathInfo")
        $xmlWriter.WriteElementString("Drive", $cmbDrives.Items[$cmbDrives.SelectedIndex].ToString())
        $xmlWriter.WriteElementString("Path", $global:currentPath)
        $xmlWriter.WriteEndElement()
        $xmlWriter.WriteEndElement()

        #write the end of the document
        $xmlWriter.WriteEndDocument()
        $xmlWriter.Flush()
        $xmlWriter.Close()
    
        [Environment]::Exit(10)
    }

}
```

The last separate function updates the current path so we default to the last used path when opening a file:

```
function setCurrentPath() {

    #note: When to change the combo box index, it will self refresh to the root of this drive. 
    #see if there is an XML file to load
    $storedPath = "F:\Scripts\SavedFileLocation.xml"
    if ( Test-Path $storedPath -PathType Leaf ) {
        #there is a file. Load it into memory
        try {
            [xml]$pathXML = Get-Content $storedPath
            #this is a very simplistic function. On an error, we just bail out
            $driveLetter = $pathXML.Root.PathInfo.Drive
            $pathInfo = $pathXML.Root.PathInfo.Path
            #if we find the drive letter in the combo box, then set the selected index to that
            foreach ($drive in $cmbDrives.Items) {
                if ( $drive -eq $driveLetter ) {
                    $cmbDrives.SelectedItem = $drive
                    $global:currentPath = $pathInfo

                }
            }
        }
        catch {
            #do nothing with the catch... just set the cmbDrives values and refresh
            $cmbDrives.SelectedIndex = 0

        } 
    }

    fillListBox($global:currentPath)

}
```

The main function doesn't do much... It essentially just sets up the form and lets the user control the rest:

```
#MAIN ROUTINE
$OpenFile.Controls.AddRange(@($lblTop, $btnCancel, $btnOK, $cmbDrives, $lstFiles, $btnUpFolder, $txtFilter, $lblFilter))

#populate the combo box
fillComboBox

setCurrentPath


#show the dialog
$OpenFile.ShowDialog()

```

