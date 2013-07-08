Test_Repo
=========
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using System.Data;
using CSharpApp.BusinessLayer;

namespace CSharpApp.Pages
{
    public partial class SecReports : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            if (!IsPostBack)
            {
                // If the user isn't logged in, then redirect them back to the standard reports screen.
                if (Session["LoggedIn"].ToString() != "true")
                {
                    Response.Redirect("Reports.aspx");
                }
                else
                {
                    // Welcome user.
                    lblWelcomeUser.Text = "Welcome " + Session["Username"] + "!";
                }

                string[] departments = { "Select a Dept","Perfumes", "Apparel", "Electronics", "Delicatessen" };
                for (int i = 0; i < departments.Length; i++)
                {
                    ddlDepartments.Items.Add(new ListItem(departments[i], i.ToString()));
                }
            } // end if

            
        }
        // Log the user out, and redirect them back to the standard reports screen (esentially, the Login screen).
        protected void btnLogout_Click(object sender, EventArgs e)
        {
            Response.Redirect("Reports.aspx");
            Session["LoggedIn"] = null;
        }

        protected void ddlDepartments_SelectedIndexChanged(object sender, EventArgs e)
        {
            UserBL us = new UserBL();
            int deptNum = Convert.ToInt32(ddlDepartments.SelectedValue);

            DataSet ds = us.populateGridview(deptNum);
            if (ds.Tables.Count > 0)
            {
                gvDept.DataSource = ds;
                gvDept.DataBind();
                btnGenerateReport.Enabled = true;
            }
            else
            {
                btnGenerateReport.Enabled = false;
               
            }
        }
    }
}
