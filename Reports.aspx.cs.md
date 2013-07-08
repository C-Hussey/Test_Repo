Test_Repo
=========
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using CSharpApp.BusinessLayer;


namespace CSharpApp
{
    public partial class Reports : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            
            lblLoginError.Visible = false;
            if (!IsPostBack)
            {
                /* If we are returning from the NewUser page, where the user has just created credentials for themselves,
                 * then accept the user info from session variables, and log the user in. */
                if (Session["Username"] != null && Session["Password"] != null)
                {
                    userLogin(Session["Username"].ToString(), Session["Password"].ToString());
                    // Null out the session variables, since we don't need them anymore.
                    Session["Username"] = null;
                    Session["Password"] = null;
                }
                // If user has a session, then they are currently logged in. Bring up the report stuff.
                if (Session["User"] != null)
                {
                    
                }
            } // end if
        } // end Page_Load

        // Redirect to a new page, to create a new user.
        protected void lbnNewUser_Click(object sender, EventArgs e)
        {
            Response.Redirect("NewUser.aspx");       
        }
        
        protected void btnLogin_Click(object sender, EventArgs e)
        {
            if (validateUser())
            {           
                UserBL us = new UserBL();
                // If user is valid, then create a session for them.
                if (us.loginUser(txtUserName.Text, txtPassword.Text))
                {
                    userLogin(txtUserName.Text, txtPassword.Text);              
                }
                else
                {
                    lblLoginError.Text = "Username or password does not exist.";
                    lblLoginError.Visible = true;
                }
            }

        } // end btnLogin_Click

        protected bool validateUser()
        {
            if(String.IsNullOrEmpty(txtUserName.Text))
            {
                lblLoginError.Text = "You must enter a username.";
                lblLoginError.Visible = true;
                return false;
            }
            else if (String.IsNullOrEmpty(txtPassword.Text))
            {
                lblLoginError.Text = "You must enter a password.";
                lblLoginError.Visible = true;
                return false;
            }

            return true;
        }

        protected void lbForgotPassword_Click(object sender, EventArgs e)
        {
            Response.Redirect("ResetPassword.aspx");
        }

        // Log in the user (create a session for them, store that session in a table).
        protected void userLogin(string username, string password)
        {
            UserBL us = new UserBL();        
            // Create a new Guid.
            Guid newGuid = System.Guid.NewGuid();
            // Assign it to a session variable.
            Session["User"] = newGuid;
            // Also, add user session to database table.
            us.enterUserSession(username, newGuid);
            Session["LoggedIn"] = "true";
            Session["Username"] = username;
            Response.Redirect("SecReports.aspx");
           
           
        }

      
    }
}
