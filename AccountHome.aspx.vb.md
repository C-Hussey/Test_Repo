Public Class AccountHome
    Inherits System.Web.UI.Page

    Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
        Dim us As New UserBL()
        'Make sure the user is logged in. If not, push them back to the Defualt page.
        If Session("LoggedIn") = Nothing Then
            Response.Redirect("Default.aspx")
        End If

        If Not IsPostBack Then
            'Get user id by username
            Dim userId As Integer = us.getUserID(Session("Username"))
            'Get Account info by user id and populate gridview.
            gvAccountInfo.DataSource = us.getAccountInfo(userId)
            gvAccountInfo.DataBind()
        End If

        'Populate dropdown list if it is empty
        If ddlOption.Items.Count = 0 Then
            Dim options As String() = {"--Select--", "Transfer funds", "Pay Bill", "Open New Account", "Cancel Account"}
            For i As Integer = 0 To options.Length - 1
                ddlOption.Items.Add(New ListItem(options(i), i.ToString()))
            Next
        End If
        
        'Rebind gvPayBill so that the gridview is refreshed if the user is ttying to pay a bill
        gvPayBill.DataSource = us.getUserBilling(Session("Username"))
        gvPayBill.DataBind()
    End Sub


    Protected Sub ddlOption_SelectedIndexChanged(sender As Object, e As EventArgs) Handles ddlOption.SelectedIndexChanged
        'Make a decision based on the selected index

        'Transfer Funds
        If ddlOption.SelectedIndex = 1 Then
            upnTransfer.Visible = True
            upnTransfer.Update()
            Dim accountNum As String = ""
            Dim amount As String = ""
            'Fill ddlActionFrom with info from the gridview
            For Each row As GridViewRow In gvAccountInfo.Rows
                accountNum = row.Cells(0).Text
                amount = row.Cells(2).Text
                ddlActionFrom.Items.Add(New ListItem("VB Bank Account # " + accountNum + " --- $" + amount, accountNum))
            Next

            'Else get the bills for the current customer
        ElseIf ddlOption.SelectedIndex = 2 Then
            Dim us As UserBL = New UserBL()
            gvPayBill.DataSource = us.getUserBilling(Session("Username"))
            gvPayBill.DataBind()
            upnPayBill.Visible = True
            upnPayBill.Update()

        ElseIf ddlOption.SelectedIndex = 3 Then

        ElseIf ddlOption.SelectedIndex = 4 Then

        End If

        upnPayBill.Update()
    End Sub

    
    Protected Sub ddlActionFrom_SelectedIndexChanged(sender As Object, e As EventArgs) Handles ddlActionFrom.SelectedIndexChanged
        Dim selectedItem As String = ddlActionFrom.SelectedItem.Text

        upnTransfer.Visible = True
        upnTransfer.Update()
        ddlActionTo.Items.Clear()
        Dim accountNum As String = ""
        Dim amount As String = ""
        'Fill ddlActionTo with info from the gridview
        For Each row As GridViewRow In gvAccountInfo.Rows
            accountNum = row.Cells(0).Text
            amount = row.Cells(2).Text
            'Exclude the account that was chosen to transfer from. No sense in transferring from Account A to Account A.
            If Not selectedItem.Contains(accountNum) Then
                ddlActionTo.Items.Add(New ListItem("VB Bank Account # " + accountNum + " --- $" + amount, accountNum))
            End If
        Next
        'Enable the Transfer button if there are selected values both in the TO and FROM dropdown lists
        If Not IsNothing(ddlActionTo.SelectedValue) Then
            btnTransfer.Enabled = True
        End If
    End Sub

    Protected Sub gvAccountInfo_SelectedIndexChanged(sender As Object, e As EventArgs)

    End Sub

    'Catch gvPayBill right after its databind and change the text in a specific column into link buttons
    Protected Sub gvPayBill_RowDataBound(sender As Object, e As GridViewRowEventArgs)
        If e.Row.RowType = DataControlRowType.DataRow Then
            Dim rowView As DataRowView = DirectCast(e.Row.DataItem, DataRowView)

            ' Retrieve the status value for the current row. 
            Dim status As String = rowView("Billing Status").ToString()
            'If the bill status is not already paid or pending, then turn this gridview element
            'into a linkbutton and add an event handler to it.
            If Not status = "Pending" And Not status = "Paid" Then
                Dim lb As New LinkButton()
                'Update the text value of the link button.
                lb.Text = status
                'Add event handlers to the newly created link buttons. Will execute LinkButton_Click on click. 
                AddHandler lb.Click, AddressOf LinkButton_Click
                'Add the link button to the gridview.
                e.Row.Cells(3).Controls.Add(lb)
            End If

        End If

    End Sub

    'Event handler. Fires when user clicks on the linkbutton in gvPayBill, and redirects them 
    'to a page where they can pay that bill.
    Protected Sub LinkButton_Click(ByVal sender As Object, ByVal e As System.EventArgs)
        Dim lb As LinkButton = DirectCast(sender, LinkButton)

        'Get the ClientId of the clicked linkbutton:
        Dim sRowIndex As String = lb.ClientID
        'Grab the last character from this id and cast to int. This will be its row identifier (the row that it's in).
        Session("SelectedRowIndex") = Convert.ToInt32(sRowIndex.Substring(sRowIndex.Length - 1))
        Dim accountName As String = gvPayBill.Rows(Session("SelectedRowIndex")).Cells(0).Text

        'Get the account/billing ID associated with that row number.
        Dim us As UserBL = New UserBL()
        Dim accountNum As Integer = us.getBillingAccountNumber(Session("Username"), accountName)
        lblPayToAccount.Text = "Paying to Account: " + accountName + " (Account Number: " + accountNum.ToString() + ")"

        Dim bankAccountNum As String = ""
        Dim amount As String = ""
        'Fill the dropdown list with info from gvAccountInfo so that we can easily see which account we are paying from, and how much
        'money is contained therein.
        If ddlPayFromAccount.Items.Count = 0 Then
            For Each row As GridViewRow In gvAccountInfo.Rows
                bankAccountNum = row.Cells(0).Text
                amount = row.Cells(2).Text
                ddlPayFromAccount.Items.Add(New ListItem("VB Bank Account # " + bankAccountNum + " --- $" + amount, bankAccountNum))
            Next
        End If

        upnPayFromAccount.Visible = True
        upnPayFromAccount.Update()
    End Sub

    Protected Sub btnSubmitPayment_Click(sender As Object, e As EventArgs) Handles btnSubmitPayment.Click
        'Validate the input amount
        If validateInput() Then
            Dim us As UserBL = New UserBL()
            Dim accountNumber As Integer = Convert.ToInt32(ddlPayFromAccount.SelectedValue)
            'Get the name of the billing account
            Dim accountName As String = gvPayBill.Rows(Session("SelectedRowIndex")).Cells(0).Text
            'Get the account number of the billing account
            Dim billingAccountNum As Integer = us.getBillingAccountNumber(Session("Username"), accountName)
            'Deduct the payment amount from the billing account by account number.
            'Updates to the appropriate tables are handled on the back end.
            If us.deductFromAccount(txtPayAmount.Text, accountNumber, billingAccountNum) Then
                'Update the Billing gridview
                gvPayBill.DataSource = us.getUserBilling(Session("Username"))
                gvPayBill.DataBind()
                'Update the Account Info gridview
                Dim userId As Integer = us.getUserID(Session("Username"))
                gvAccountInfo.DataSource = us.getAccountInfo(userId)
                gvAccountInfo.DataBind()

                'Clear and reload the dropdown list to reflect any changes to accounts
                ddlPayFromAccount.Items.Clear()
                For Each row As GridViewRow In gvAccountInfo.Rows
                    Dim bankAccountNum As String = row.Cells(0).Text
                    Dim amount As String = row.Cells(2).Text
                    ddlPayFromAccount.Items.Add(New ListItem("VB Bank Account # " + bankAccountNum + " --- $" + amount, bankAccountNum))
                Next

                'Remove the update panel, thereby hiding all the associated controls
                upnPayFromAccount.Visible = False
            End If
        End If
    End Sub

    'Ensure that no erroneous text was typed into the textbox.
    Protected Function validateInput() As Boolean

        If Not IsNumeric(txtPayAmount.Text) Then
            lblPayError.Text = "**Input amount was not a number or was not in the correct format"
            lblPayError.Visible = True
            Return False
        End If
        lblPayError.Visible = False
        Return True
    End Function

    'If user clicks Cancel, then re-hide the update panel
    Protected Sub btnCancel_Click(sender As Object, e As EventArgs) Handles btnCancel.Click
        upnPayFromAccount.Visible = False
    End Sub

    Protected Sub btnTransfer_Click(sender As Object, e As EventArgs) Handles btnTransfer.Click
        Dim fromAccount As String = ddlActionFrom.SelectedValue
        Dim toAccount As String = ddlActionTo.SelectedValue

        Dim us As UserBL = New UserBL()
        'Deduct from the first acount
        us.deductFromAccount(txtTransferAmount.Text, fromAccount)
        'Add to the second account
        us.addToAccount(txtTransferAmount.Text, toAccount)

        'Update the account info gridview to reflect the changes.
        Dim userId As Integer = us.getUserID(Session("Username"))
        gvAccountInfo.DataSource = us.getAccountInfo(userId)
        gvAccountInfo.DataBind()

        'Update the dropdown list to reflect the changes.
        ddlActionFrom.Items.Clear()
        For Each row As GridViewRow In gvAccountInfo.Rows
            Dim bankAccountNum As String = row.Cells(0).Text
            Dim amount As String = row.Cells(2).Text
            ddlActionFrom.Items.Add(New ListItem("VB Bank Account # " + bankAccountNum + " --- $" + amount, bankAccountNum))
        Next

        'Update the dropdown list to reflect the changes.
        ddlActionTo.Items.Clear()
        'For Each row As GridViewRow In gvAccountInfo.Rows
        '    Dim bankAccountNum As String = row.Cells(0).Text
        '    Dim amount As String = row.Cells(2).Text
        '    ddlActionTo.Items.Add(New ListItem("VB Bank Account # " + bankAccountNum + " --- $" + amount, bankAccountNum))
        'Next

    End Sub
End Class
