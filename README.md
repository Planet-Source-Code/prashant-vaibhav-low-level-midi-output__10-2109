<div align="center">

## Low Level MIDI Output


</div>

### Description

.NET has access to the full DirectX9 API, with all its goodies related to Audio. But when it comes to MIDI, .NET is a no-go. However, with a few unmanaged Win32 API calls, you can make low level, individual message-based MIDI output possible for your own projects. This little sample from Prashant's upcoming "MIDI Control for .NET Framework" shows you how. It was developed in SharpDevelop.

It encapsulates the function-based MIDI output mechanism of Windows into a convenient class which can be used directly in any of your projects. The code takes care of the ugly details for marshalling, opening, closing and managing device handles etc.

To use it, just copy it into a separate file and add the file to your project. Then Import the namespace "Prashant.MIDIControl".
 
### More Info
 
You need to know the three byte MIDI message codes to make any kind of sounds!

Note that this example uses unmanaged dll calls, so there may be issues that I'm unaware of, however in my testing nothing has been discovered.


<span>             |<span>
---                |---
**Submitted On**   |
**By**             |[Prashant Vaibhav](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByAuthor/prashant-vaibhav.md)
**Level**          |Intermediate
**User Rating**    |4.6 (37 globes from 8 users)
**Compatibility**  |VB\.NET
**Category**       |[Graphics/ Sound](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByCategory/graphics-sound__10-15.md)
**World**          |[\.Net \(C\#, VB\.net\)](https://github.com/Planet-Source-Code/PSCIndex/blob/master/ByWorld/net-c-vb-net.md)
**Archive File**   |[](https://github.com/Planet-Source-Code/prashant-vaibhav-low-level-midi-output__10-2109/archive/master.zip)

### API Declarations

```
Copyright (C) 2004 by Prashant Vaibhav.
All the code material is only for educational purposes!
```


### Source Code

```
Win32Wrapper.vb
===========================================
' Prashant's MIDI Control for .NET Framework
'
' Copyright © 2004 by Prashant Vaibhav
'
' Code in this file can only be used for educational purposes
'
' Created on 23-Feb-2004 at 19:43 using SharpDevelop
' Wraps access to the unmanaged Win32 API functions
Imports System.Runtime.InteropServices
Namespace Prashant.MIDIControl
	'This structure holds info about a MIDI out device
	<StructLayout(LayoutKind.Sequential, CharSet := CharSet.Auto)> _
	Public Structure MIDIOUTCAPS
		Dim ManufacturerID As Short
		Dim ProductID As Short
		Dim DriverVersion As Integer
		<MarshalAs(UnmanagedType.ByValTStr, SizeConst := 32)> _
		Dim Label As String
		Dim Technology As Short
		Dim Voices As Short
		Dim Notes As Short
		Dim ChannelMask As Short
		Dim Support As Integer
	End Structure
	'Following class contains all the win32 api functions needed for MIDI output
	Friend Class Win32API
		Declare Function midiOutGetNumDevs Lib "winmm.dll" () As Integer
		Declare Auto Function midiOutGetDevCaps Lib "winmm.dll" (ByVal DevNum As Integer, ByRef DevCaps As MIDIOUTCAPS, ByVal SizeOfStruc As Integer) As Integer
		Declare Function midiOutOpen Lib "winmm.dll" (ByRef hDev As Integer, Byval devID As Integer, ByVal cbfunc As Integer, ByVal cbdata As Integer, ByVal cboptions As Integer) As Integer
		Declare Sub midiOutClose Lib "winmm.dll"(ByVal hdev As Integer)
		Declare Function midiOutShortMsg Lib "winmm.dll"(ByVal hdev As Integer, ByVal msg As Integer) as integer
	End Class
End NameSpace
========================================
BaseImplementation.vb
========================================
' Prashant's MIDI Control for .NET Framework
'
' Copyright © 2004 by Prashant Vaibhav
'
' Code in this file can only be used for educational purposes
'
' Created on 23-Feb-2004 at 21:20 using SharpDevelop
' Implements the actual MIDI Output mechanism
Imports Prashant.MIDIControl
Imports System.Runtime.InteropServices
Namespace Prashant.MIDIControl
	Public Class MIDIOutputPort
		Private hDev As Integer 'Handle to the midi device
		Public Info As MIDIOUTCAPS
		Public Opened As Boolean
		Public Sub New(Optional ByVal DevID As Integer = -1)
			'Here we open the MIDI device
			'Note that if the device ID is not given, -1 is used
			'Because -1 is the MIDI Mapper (the default MIDI device set in control panel)
			hDev = 0 : Opened = False
			'Lets store the information about this MIDI device
			Win32API.midiOutGetDevCaps(DevID, Info, Marshal.SizeOf(Info))
			'And now open it
			Opened = Not Win32API.midiOutOpen(hDev, devid, 0, 0, 0)
			'^-- Opened will be true if the device was successfully opened, else false
		End Sub
		Protected Overrides Sub Finalize()
			'When the class instance is destroyed, we MUST close the device
			'Else it will be unusable until after a reboot!
			If hDev <> 0 Then Win32API.midiOutClose(hDev)
		End Sub
		Public ReadOnly Property NumDevices As Integer
			'Read only property that returns the no. of MIDI output devices installed in the system
			Get
				Return Win32API.midiOutGetNumDevs
			End Get
		End Property
		Public Sub SendMsg(ByVal Status As Byte, Data1 As Byte, Data2 As Byte)
			'This method will allow us to send a short 3 byte MIDI message
			Dim msg As Integer
			'Calculate the 32-byte message from the three bytes
			msg = Data2 * 65536 + Data1 * 256 + Status
			'And send it to the device
			If hDev<>0 Then Win32API.midiOutShortMsg(hDev, msg)
		End Sub
	End Class
End NameSpace
============================================
DemoApp.vb - demonstrates how to use the class
==============================================
' Prashant's MIDI Control for .NET Framework
'
' Copyright © 2004 by Prashant Vaibhav
'
' Code in this file can only be used for educational purposes
' Demo Form. Plays a snare drum when the button is clicked.
' Created on 23-Feb-2004 at 21:46 using SharpDevelop
Imports System
Imports System.Drawing
Imports System.Windows.Forms
Imports Prashant.MIDIControl
Namespace Prashant.MIDIControl.Demos
	Public Class PlaySnare
		Inherits System.Windows.Forms.Form
		Private button As System.Windows.Forms.Button
		Public Shared Sub Main
			Application.Run(New PlaySnare())
		End Sub
		'We will use the following object to access the MIDI output port
		Private Midi As MIDIOutputPort
		Public Sub New()
			MyBase.New
			' Must be called for initialization
			Me.InitializeComponent
			Midi = New MIDIOutputPort 'This opens the MIDI port and makes it ready for use
			'Note that you can open any other device by specifying its ID while creating the object
			'e.g. Midi = New MIDIOutputPort(3) will open the third device
			'You can get no. of devices from Midi.NumDevices
			Me.Text = Midi.Info.Label
		End Sub
		Private Sub InitializeComponent()
			Me.button = New System.Windows.Forms.Button
			Me.SuspendLayout
			'
			'button
			'
			Me.button.Location = New System.Drawing.Point(80, 32)
			Me.button.Name = "button"
			Me.button.Size = New System.Drawing.Size(120, 24)
			Me.button.TabIndex = 0
			Me.button.Text = "Play Snare Drum"
			AddHandler Me.button.MouseUp, AddressOf Me.ButtonMouseUp
			AddHandler Me.button.MouseDown, AddressOf Me.ButtonMouseDown
			'
			'PlaySnare
			'
			Me.AutoScaleBaseSize = New System.Drawing.Size(5, 13)
			Me.ClientSize = New System.Drawing.Size(272, 101)
			Me.Controls.Add(Me.button)
			Me.FormBorderStyle = System.Windows.Forms.FormBorderStyle.FixedSingle
			Me.MaximizeBox = false
			Me.Name = "PlaySnare"
			Me.Text = "Prashant's MIDI Control - Demo"
			Me.ResumeLayout(false)
		End Sub
	Private Sub ButtonMouseDown(sender As Object, e As System.Windows.Forms.MouseEventArgs)
		'99 is "note on on channel 10" (9+1 = 10 = drums channel)
		'38 is the note number for a snare drum
		'127 means maximum velocity (loudness)
		Midi.SendMsg(&H99, 38, 127)
		'You can find these codes from the internet (www.midi.org)
	End Sub
	Private Sub ButtonMouseUp(sender As Object, e As System.Windows.Forms.MouseEventArgs)
		'Send the same command note number but with velocity = 0
		'..to turn that note off (stop sounding)
		Midi.SendMsg(&H99,38,0)
	End Sub
	End Class
End Namespace
```

