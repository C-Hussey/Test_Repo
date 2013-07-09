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
        // User has clicked buton o reset password.
        protected void btnConfirm_Click(object sender, EventArgs e)
        {
            resetLabels();
            if (validateForm())
            {

                UserBL us = new UserBL();
                try
                {

                    // Check if username exists in database.
                    if (us.usernameExists(txtUsername.Text) == null)
                    {
                        // Check that the new password and the confirm new pasword matches.
                        if (!txtNewPassword.Text.Equals(txtConfirmPassword.Text))
                        {
                            lblResult.Text = "Passwords does not match.";
                            lblNewPasswordErr.Visible = true;
                            lblConfirmPasswordErr.Visible = true;
                            return;
                        }
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
                            btnCancel.Enabled = false;
                            btnConfirm.Enabled = false;
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
            }

        } // end lbnConfirm_Click

        protected void btnCancel_Click(object sender, EventArgs e)
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
            lblConfirmPasswordErr.Visible = false;
            lblNewPasswordErr.Visible = false;
        }
        protected void resetLabels()
        {
            lblConfirmPasswordErr.Visible = false;
            lblNewPasswordErr.Visible = false;
            lblOldPasswordErr.Visible = false;
            lblUsernameErr.Visible = false;
        }
        // User if finished updating. Redirect back to reports page.
        protected void lbnOK_Click(object sender, EventArgs e)
        {
            Response.Redirect("Reports.aspx");
        }
        // Validate the form.
        protected bool validateForm()
        {
            // If the username field is left blank:
            if (String.IsNullOrEmpty(txtUsername.Text) || String.IsNullOrWhiteSpace(txtUsername.Text))
            {
                lblResult.Text = "Username must be supplied.";
                lblUsernameErr.Visible = true;
                return false;
            }
            // Else if the old password field is left blank:
            else if (String.IsNullOrEmpty(txtOldPassword.Text) || String.IsNullOrWhiteSpace(txtOldPassword.Text))
            {
                lblResult.Text = "Previous password must be supplied.";
                lblOldPasswordErr.Visible = true;
                return false;
            } 
     
            // Else if the new password field is left blank:
            else if (String.IsNullOrEmpty(txtNewPassword.Text) || String.IsNullOrWhiteSpace(txtNewPassword.Text))
            {
                lblResult.Text = "Password must be supplied.";
                lblNewPasswordErr.Visible = true;
                return false;
            }
            // Else if the confirm password field is left blank:
            else if (String.IsNullOrEmpty(txtConfirmPassword.Text) || String.IsNullOrWhiteSpace(txtConfirmPassword.Text))
            {
                lblResult.Text = "Password must be supplied.";
                lblConfirmPasswordErr.Visible = true;
                return false;
            }
            // Else if the password is not long enough:
            else if (txtNewPassword.Text.Length < 6)
            {
                lblResult.Text = "Password must be at least 6 characters in length.";
                lblNewPasswordErr.Visible = true;
                return false;
            }
            // Else if the passwords do not match:
            else if (!txtNewPassword.Text.Equals(txtConfirmPassword.Text))
            {
                lblResult.Text = "Passwords do not match.";
                lblConfirmPasswordErr.Visible = true;
                lblNewPasswordErr.Visible = true;
                return false;
            }

            return true;
        }

    } // end class
}
