Imports System.Data.SqlClient

Public Class Validation
    'Ensure that no erroneous text was typed into the textbox.
    Public Function validateInput(ByVal inputText As String, ByRef label As Label) As Boolean

        If Not IsNumeric(inputText) Then
            label.Text = "**Input amount was not a number or was not in the correct format"
            label.Visible = True
            Return False
        ElseIf Convert.ToDecimal(inputText) < 0 Then
            label.Text = "Value cannot be negative"
            label.Visible = True
            Return False
        End If
        'label.Visible = False
        Return True
    End Function

    Public Function validateBilling(ByVal billingAcctNum As Integer, ByVal paymentAmount As Decimal) As Boolean

        Dim dataset As New DataSet()
        Dim us As UserBL = New UserBL()

        Try

            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("ValidateBilling", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@AccountNumber", billingAcctNum))

            Dim da As New SqlDataAdapter(cmd)

            da.Fill(dataset)

            'Get the amount from the account
            Dim amount As Decimal = dataset.Tables(0).Rows(0)("BillAmount")
            'If the the amount is less than the payment amount, then reject it. Disallows users overpaying on a bill.
            If amount < paymentAmount Then
                Return False
            End If

        Catch e As Exception
            Console.WriteLine(e.Message)
            Return False
        End Try

        Return True

    End Function
End Class
