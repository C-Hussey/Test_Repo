Imports System.Data.SqlClient

Public Class UserBL
    'Search for user in aspnetdb, as well as in a table specific to this application.
    Public Function validateUser(ByVal username As String, ByVal password As String) As String
        Try
            If Not Membership.ValidateUser(username, password) Then
                Return "Username or password is invalid"
            End If
        Catch ex As Exception
            Return ex.Message
        End Try

        Return Nothing
    End Function

    Public Function getAccountInfo(ByVal userID As String) As DataSet
        Dim dataset As New DataSet()
        Try
            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("GetBankAccountInfoByUserID", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@UserID", userID))

            Dim da As New SqlDataAdapter(cmd)

            da.Fill(dataset)
        Catch e As Exception

            Console.WriteLine(e.Message)
        End Try

        Return dataset
    End Function

    Public Function getUserID(ByVal username As String) As Integer
        Dim dataset As New DataSet()
        Try
            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("GetUserIDByUsername", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@Username", username))

            Dim da As New SqlDataAdapter(cmd)

            da.Fill(dataset)

        Catch e As Exception

            Console.WriteLine(e.Message)
        End Try

        Try
            Return dataset.Tables(0).Rows(0)("UserID")
        Catch ex As Exception
            Return Nothing
        End Try

    End Function

    Public Function getUserBilling(ByVal username As String) As DataSet
        Dim dataset As New DataSet()
        Dim us As UserBL = New UserBL()

        Try
            Dim userId As Integer = us.getUserID(username)

            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("GetBillingByUserID", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@UserID", userId))

            Dim da As New SqlDataAdapter(cmd)

            da.Fill(dataset)

        Catch e As Exception

            Console.WriteLine(e.Message)
        End Try

        Return dataset

    End Function

    Public Function getBillingAccountNumber(ByVal username As String, ByVal accountName As String) As Integer
        Dim dataset As New DataSet()
        Dim us As UserBL = New UserBL()

        Try
            Dim userId As Integer = us.getUserID(username)

            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("GetBillingAccountNumber", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@UserID", userId))
            cmd.Parameters.Add(New SqlParameter("@AccountName", accountName))

            Dim da As New SqlDataAdapter(cmd)

            da.Fill(dataset)

        Catch e As Exception
            Console.WriteLine(e.Message)
        End Try

        Return dataset.Tables(0).Rows(0)("AccountNumber")

    End Function

    Public Function deductFromAccount(ByVal deduction As Decimal, ByVal accountNumber As Decimal, ByVal userID As Integer, Optional ByVal billingAccountNum As Decimal = Nothing) As Boolean
        Dim dataset As New DataSet()
        Dim us As UserBL = New UserBL()

        Try

            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("DeductFromAccount", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@BillingAccountNum", billingAccountNum))
            cmd.Parameters.Add(New SqlParameter("@AccountNumber", accountNumber))
            cmd.Parameters.Add(New SqlParameter("@Deduction", deduction))
            cmd.Parameters.Add(New SqlParameter("@UserID", userID))
            cmd.ExecuteNonQuery()
        Catch e As Exception
            Console.WriteLine(e.Message)
            Return False
        End Try

        Return True

    End Function

    Public Function addToAccount(ByVal sum As Decimal, ByVal accountNumber As Decimal) As Boolean
        Dim dataset As New DataSet()
        Dim us As UserBL = New UserBL()

        Try

            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("AddToAccount", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@AccountNumber", accountNumber))
            cmd.Parameters.Add(New SqlParameter("@Sum", sum))

            cmd.ExecuteNonQuery()
        Catch e As Exception
            Console.WriteLine(e.Message)
            Return False
        End Try

        Return True

    End Function

    'Make sure the account is not overdrawn by a transfer of funds.
    Public Function validateFunds(ByVal accountNumber As Integer, ByVal transferAmount As Decimal) As Boolean
        Dim dataset As New DataSet()
        Dim us As UserBL = New UserBL()

        Try

            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("ValidateFunds", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@AccountNumber", accountNumber))

            Dim da As New SqlDataAdapter(cmd)

            da.Fill(dataset)

            'Get the amount from the account
            Dim amount As Decimal = dataset.Tables(0).Rows(0)("Amount")
            'If the amount in the account is less than the amount the user is trying to transfer, then return false.
            If amount < transferAmount Then
                Return False
            End If

        Catch e As Exception
            Console.WriteLine(e.Message)
            Return False
        End Try

        Return True

    End Function

    Public Function createDefaultAccountInfo(ByVal username As String) As Boolean
        Dim dataset As New DataSet()
        Dim us As UserBL = New UserBL()

        Try

            Dim conn As SqlConnection = Nothing
            conn = New SqlConnection(System.Configuration.ConfigurationManager.ConnectionStrings("ApplicationServices").ConnectionString)
            conn.Open()

            Dim cmd As New SqlCommand("CreateDefaultAccountInfo", conn)
            cmd.CommandType = CommandType.StoredProcedure

            cmd.Parameters.Add(New SqlParameter("@Username", username))

            cmd.ExecuteNonQuery()

            Return True

        Catch e As Exception
            Console.WriteLine(e.Message)
            Return False
        End Try

        Return False
    End Function
End Class
