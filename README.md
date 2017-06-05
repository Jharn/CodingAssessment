# CodingAssessment
Project Database Code
Imports System.Data.OleDb
Public Class SignInPage
    Inherits System.Web.UI.Page
    Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load

    End Sub

    Public Class GlobalVariables
        'Global variable that stores the failed attempts as an integer.
        Public Shared fails As Integer = 0
    End Class


    Protected Sub btnVerify_Click(sender As Object, e As EventArgs) Handles btnVerify.Click
        'grabs the connectionString from web.config file
        Dim AccessCS As String = ConfigurationManager.ConnectionStrings("AccessCS").ConnectionString

        'Create a connection variable to the database.
        Dim con As OleDbConnection
        'Assign to connectionString to find the database
        con = New OleDbConnection(AccessCS)

        Try
            'Creates a datareader that is used to read data from database
            Dim dr As OleDbDataReader
            'Create SQL command that specifies exactly how to read the data.
            Dim cmd As New OleDbCommand

            With cmd
                'selects email and password from the players table in database
                .CommandText = "SELECT * FROM Players WHERE Email =  '" & txtUserID.Text & "' AND Password =  '" & txtPassword.Text & "'"
                'this tells command to use connection to the database.
                .Connection = con
            End With

            'this code opens a connection to the database
            con.Open()

            'Executes the SQL command from earlier and grabs the data from the select statement (for Email and Password).
            dr = cmd.ExecuteReader(CommandBehavior.CloseConnection)

            'Declared variable that contains the number of rows that have been successfully read from the query response. 
            'If it still kept its original value, i.e. 0, then it's only fair to assume that nothing has been read -
            '(or rather, there is nothing to read) from the query response.
            Dim RowsRead As Integer = 0
            'Loops through rows and records found by datareader
            While dr.Read
                'Stores the email and password grabbed by the datareader into seperate strings.
                Dim email As String = (dr("Email"))
                Dim pw As String = (dr("Password"))
                'Checks if the textbox input matches the stored Email and Password strings from the database.
                If (dr("Email")) = txtUserID.Text And (dr("Password")) = txtPassword.Text Then
                    lblLoginMessage.Text = "Found name :" & email
                    lblPass.Text = "Password Found: " & pw
                End If
                'Add 1 to RowsRead if no data has been matched/read.
                RowsRead += 1
                'Reads each character of the password, if one is Numeric set hasNumber to true and exit loop.
                Dim hasNumber As Boolean = False
                Dim input As String = pw
                Dim characters() As Char = input
                For Each c As Char In characters
                    If IsNumeric(c) Then
                        hasNumber = True
                        Exit For
                    End If
                Next
                'Displays if the password contains a number using "true" or "false" boolean.
                lblNumeric.Text = "Password contains number: " & hasNumber
            End While




            'If nothing from the datareader has been read execute this code:
            If (RowsRead = 0) Then
                'Prompt the user that there is no record found for the Email/Password entered
                lblLoginMessage.Text = "Email: " & txtUserID.Text & " not found in database with matching password, please try again."
                lblPass.Text = "Password: " & txtPassword.Text & " not found in database with matching email, please try again."
                'Add one to the fail attempts named "fails"
                GlobalVariables.fails += 1
                'Prompt the user that they have failed + 1 login attempt
                lblCount.Text = "You have failed to log in " + Str(GlobalVariables.fails) + " times."
                'Clear the isNumeric label.
                lblNumeric.Text = "Password contains number: "
            End If

            'Checks if any textboxes are left blank, if so, prompts the user to enter a Email or Password and clears all other boxes/labels.
            If txtUserID.Text = "" Or txtPassword.Text = "" Then
                lblError.Text = "One of the following fields is blank: " & " Email Or" & " Password."
                txtUserID.Text = ""
                txtPassword.Text = ""
                lblLoginMessage.Text = "Email: "
                lblPass.Text = "Password: "
                lblNumeric.Text = "Password contains number: "
            Else
                'Clears error message if everything is entered correctly a second time round.
                lblError.Text = ""
            End If

            'If the user fails 5 login attempts this clears all the textboxes and disables the login button. 
            'It also displays the error message:  "You have failed your login 5 times and have been locked out."
            If GlobalVariables.fails = 5 Then
                btnVerify.Enabled = False
                lblCount.Text = ""
                lblLoginMessage.Text = "Email: "
                lblPass.Text = "Password: "
                lblError.ForeColor = Drawing.Color.Red
                lblError.Text = "You have failed your login 5 times and have been locked out."
                lblNumeric.Text = "Password contains number: "
            End If

            'Catches any errors and outputs them on a label.
        Catch ex As Exception
            lblLoginMessage.Text = "Error found :" & ex.Message
        Finally
            'Closes connection to database.
            con.Close()
        End Try

    End Sub
End Class
