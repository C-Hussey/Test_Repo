Test_Repo
=========
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using CSharpApp.BusinessLayer;

namespace CSharpApp.Pages
{
    public partial class ResetPassword : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            if(!IsPostBack){
                resetFields();  
            }
        }

        protected void lbnConfirm_Click(object sender, EventArgs e)
        {
            UserBL us = new UserBL();
            try
            {
              
                // Check if username exists in database.
                if (us.usernameExists(txtUsername.Text) == null)
                {
                    // Check that the old password matches the input, and change the password.
                    string result = us.updatePassword(txtUsername.Text, txtOldPassword.Text, txtNewPassword.Text);
                    // A null result equates to sucess. If result hold a string message, then there was an error somewhere.
                    if (result == null)
                    {
                        // Display success message.
                        lblResult.Text = "Password successfully updated.";
                        // Reset text fields.
                        txtNewPassword.Text = "";
                        txtOldPassword.Text = "";
                        txtUsername.Text = "";
                        txtConfirmPassword.Text = "";
                        lbnOK.Visible = true;
                        lbnCancel.Enabled = false;
                        lbnConfirm.Enabled = false;
                    }
                    // Else output whatever the error message is.
                    else
                    {
                        lblResult.Text = result;
                    }
                } // end if
            }
            catch (Exception)
            {
                throw new Exception("Unable to update password");
            }
             
        } // end lbnConfirm_Click

        protected void lbnCancel_Click(object sender, EventArgs e)
        {
            Response.Redirect("Reports.aspx");
        }

        protected void resetFields()
        {
            lblResult.Text = "";
            txtNewPassword.Text = "";
            txtOldPassword.Text = "";
            txtUsername.Text = "";
            txtConfirmPassword.Text = "";
            lbnOK.Visible = false;
        }

        protected void lbnOK_Click(object sender, EventArgs e)
        {
            Response.Redirect("Reports.aspx");
        }
    }
}
