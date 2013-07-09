using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using CSharpApp.BusinessLayer;

namespace CSharpApp.Account
{
    public partial class NewUser : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            resetFields();
        }

        // Attempt to create a new user.
        protected void btnConfirm_Click(object sender, EventArgs e)
        {
            // Validate that all the fields are valid and filled out correctly.
            if (validateForm())
            {
                // Create new user.
                UserBL us = new UserBL();
                string result = null;

                result = us.insertNewUser(txtUsername.Text, txtPassword.Text, txtEmail.Text);
                // If string result is not null, then it is returning an error message. Output that error.
                if (!String.IsNullOrEmpty(result))
                {
                    lblResult.Text = result;
                    return;
                } // end if    
                // Else the user was able to be created. Fill some session variables, and redirect back to reports page.
                else
                {
                    Session["Username"] = txtUsername.Text;
                    Session["Password"] = txtPassword.Text;
                    Response.Redirect("Reports.aspx");         
                }
            } // end if
        } // end lbnConfirm_Click

        protected bool validateForm()
        {
            // If the username field is left blank:
            if (String.IsNullOrEmpty(txtUsername.Text) || String.IsNullOrWhiteSpace(txtUsername.Text))
            {
                lblResult.Text = "Username must be supplied.";
                lblUsernameErr.Visible = true;
                return false;
            }
            // Else if the email field is left blank:
            else if (String.IsNullOrEmpty(txtEmail.Text) || String.IsNullOrWhiteSpace(txtEmail.Text))
            {
                lblResult.Text = "Email address must be supplied.";
                lblEmailErr.Visible = true;
                return false;
            } // check for email validity
            // Else if the email address is not in a valid form:
            else if (!validateEmail())
            {
                lblResult.Text = "Email address is not valid.";
                lblEmailErr.Visible = true;
                return false;
            }
            // Else if the password field is left blank:
            else if (String.IsNullOrEmpty(txtPassword.Text) || String.IsNullOrWhiteSpace(txtPassword.Text))
            {
                lblResult.Text = "Password must be supplied.";
                lblPasswordErr.Visible = true;
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
            else if (txtPassword.Text.Length < 6)
            {
                lblResult.Text = "Password must be at least 6 characters in length.";
                lblPasswordErr.Visible = true;
                return false;
            }
            // Else if the passwords do not match:
            else if (!txtPassword.Text.Equals(txtConfirmPassword.Text))
            {
                lblResult.Text = "Passwords do not match.";
                lblConfirmPasswordErr.Visible = true;
                lblPasswordErr.Visible = true;
                return false;
            }

            return true;
        }

        // User clicked "Cancel". Redirect back to the previous page.
        protected void btnCancel_Click(object sender, EventArgs e)
        {
            Response.Redirect("Reports.aspx");
        }

        /* Validate email address. Code taken from http://stackoverflow.com/questions/1365407/c-sharp-code-to-validate-email-address.
         * There is some debate as to whether or not this handles all cases of email validation, but for the purposes of this sample application,
         * it is good enough. */ 
        protected bool validateEmail()
        {
            try
            {
                var addr = new System.Net.Mail.MailAddress(txtEmail.Text);
                return true;
            }
            catch
            {
                return false;
            }
        }

        protected void resetFields()
        {
            lblConfirmPasswordErr.Visible = false;
            lblEmailErr.Visible = false;
            lblPasswordErr.Visible = false;
            lblUsernameErr.Visible = false;
        }
    } // end class
}
