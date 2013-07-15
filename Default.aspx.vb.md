Public Class _Default
    Inherits System.Web.UI.Page

    Protected Sub Page_Load(ByVal sender As Object, ByVal e As System.EventArgs) Handles Me.Load
        If Not IsPostBack Then
            resetValidationFields()
        End If
    End Sub

    Protected Sub btnLogin_Click(sender As Object, e As EventArgs) Handles btnLogin.Click
        If validateLogin() Then
            Dim us As New UserBL()
            Dim result As String = us.validateUser(txtUsername.Text, txtPassword.Text)
            If Not IsNothing(result) Then
                lblLoginErrMsg.Text = result
                lblLoginErrMsg.Visible = True
                lblPasswordErr.Visible = True
                lblUsernameErr.Visible = True
            Else
                Session("LoggedIn") = True
                Session("Username") = txtUsername.Text
                Response.Redirect("Pages/AccountHome.aspx")
            End If
        End If
    End Sub

    Protected Function validateLogin() As Boolean
        If String.IsNullOrEmpty(txtPassword.Text) Then
            lblLoginErrMsg.Text = "Password field cannot be empty."
            lblLoginErrMsg.Visible = True
            lblPasswordErr.Visible = True
            Return False
        ElseIf String.IsNullOrEmpty(txtUsername.Text) Then
            lblLoginErrMsg.Text = "Username field cannot be empty."
            lblLoginErrMsg.Visible = True
            lblUsernameErr.Visible = True
            Return False
        End If
        Return True
    End Function

    Protected Function resetValidationFields() As Object
        lblPasswordErr.Visible = False
        lblUsernameErr.Visible = False
        lblLoginErrMsg.Visible = False
    End Function


    Protected Sub lbnCreateAcct_Click(sender As Object, e As EventArgs) Handles lbnCreateAcct.Click
        Response.Redirect("Account/Register.aspx")
    End Sub

   
    Protected Sub lbABout_Click(sender As Object, e As EventArgs) Handles lbABout.Click
        Response.Redirect("About.aspx")
    End Sub

    Protected Sub lbLinks_Click(sender As Object, e As EventArgs) Handles lbLinks.Click
        Response.Redirect("http://cjhdemo.com/entry")
    End Sub
End Class
