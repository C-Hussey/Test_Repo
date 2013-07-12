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
                try
                {

                    // If the user isn't logged in, then redirect them back to the standard reports screen.
                    if (Session["LoggedIn"].ToString() == null)
                    {
                        Response.Redirect("Reports.aspx");
                    }
                    else
                    {
                        // Welcome user.
                        lblWelcomeUser.Text = "Welcome " + Session["Username"] + "!";
                        txtUserName.Enabled = false;
                        txtPassword.Enabled = false;
                    }
                }
                catch (Exception)
                {
                    Response.Redirect("Reports.aspx");
                }

                populateDropdownList();
                populateCheckBoxList();

                // If the gridview is empty, disable any other controls that perform actions on it.
                if (gvDept.Columns.Count > 0)
                {
                    enableGridviewDependants();
                }
                else
                {
                    disableGridviewDependants();
                }
            } // end if      

            // Hide or show gridview columns, based on the user's selections.
            hideShowGridviewElements();
               
        }
        /* Log the user out, delete their session and redirect them back to the standard reports screen 
        (esentially, the Login screen).*/
        protected void btnLogout_Click(object sender, EventArgs e)
        {
            UserBL us = new UserBL();
            // Delete the user session that was stored in a database table.
            us.deleteUserSession(Session["Username"].ToString(), Guid.Parse(Session["User"].ToString()));
            // Remove all session variables.
            clearAllSessions();
            // Redirect to self and let Page Load push user back to the normal reports page.
            Response.Redirect("SecReports.aspx");
            
        }
        // Get information from db to populate the gridview with the appropritate departmental info.
        protected void ddlDepartments_SelectedIndexChanged(object sender, EventArgs e)
        {
            UserBL us = new UserBL();
            int deptNum = Convert.ToInt32(ddlDepartments.SelectedValue);
            // Returns either a result set filtered by deptID, or all possible results. Dependent on user selection.
            DataSet ds = determineOptionalParam(deptNum);
            // If user goes back to the 0th element in the dropdown list, disable controls.
            if (ddlDepartments.SelectedIndex == 0)
            {
                disableGridviewDependants();
            }  
            // Else bind results to the gridview and enable controls.
            else if(ds.Tables.Count > 0)
            {
                gvDept.DataSource = ds;               
                gvDept.DataBind();
                enableGridviewDependants();
            }
            else
            {
                disableGridviewDependants();
               
            }
        } // end method

        // Call when user is logging out. Deletes all sessions created during the login process.
        protected void clearAllSessions()
        {
            Session.Remove("LoggedIn");
            Session.Remove("Username");
            Session.Remove("Password");
            Session.Remove("User"); 
         
        }
        // Dynamically populate dropdown list with some info.
        protected void populateDropdownList()
        {
            string[] departments = { "--Select a Dept--", "All Depts", "Perfumes", "Apparel", "Electronics", "Delicatessen" };
                for (int i = 0; i < departments.Length; i++)
                {
                    ddlDepartments.Items.Add(new ListItem(departments[i], i.ToString()));
                }
        }
        // Dynamically populate checkboxlist with some info.
        protected void populateCheckBoxList()
        {
            string[] views = { "Start/End Date", "Weekly Schedule", "Total Hours worked", "Pay Rate", "Pay Type" };
                for (int i = 0; i < views.Length; i++)
                {
                    ListItem item = new ListItem(views[i], i.ToString());
                    cblEmployeeView.Items.Add(item);
                    // Check each checkbox by default.
                    item.Selected = true;
                }
        }
        // Enable controls that perform actions on the gridview.
        protected void enableGridviewDependants()
        {
            btnGenerateReport.Enabled = true;
            cblEmployeeView.Enabled = true;
        }
        // Disable controls that perform actions on the gridview.
        protected void disableGridviewDependants()
        {
            btnGenerateReport.Enabled = false;
            cblEmployeeView.Enabled = false;
        }

        // Hide or display gridview columns, based on user selection in the checkboxlist.
        protected void hideShowGridviewElements()
        {   
            int counter = 0;
            bool boolVal = false;
            // For each checkbox in the checkboxlist, set boolean to true (checked) or false (unchecked).
            foreach (ListItem item in cblEmployeeView.Items)
            {
                if (!item.Selected && cblEmployeeView.Enabled)
                {
                    boolVal = false;
                }
                else
                {
                    boolVal = true;
                }
                    // Switch statement. "counter" represents the relative index of "item".
                    switch (counter)
                    {
                        case 0:// If Start/End Date is not selected, then hide columns 3 and 4
                            showHide(3,boolVal);
                            showHide(4,boolVal);
                            break;
                        case 1:// If Weekly Schedule is not selected, then hide columns 5-11
                            showHide(5,boolVal);
                            showHide(6,boolVal);
                            showHide(7,boolVal);
                            showHide(8,boolVal);
                            showHide(9,boolVal);
                            showHide(10,boolVal);
                            showHide(11,boolVal);
                            break;
                        case 2:// If Total Hours worked is not selected, then hide column 12
                            showHide(12,boolVal);
                            break;
                        case 3:// If Pay Rate is not selected, then hide column 13
                            showHide(13,boolVal);
                            break;
                        case 4:// If Pay Type is not selected, then hide column 14
                            showHide(14,boolVal);
                            break;
                    } // end switch   
                    // increment counter
                    counter++;
                }
            }

        // Hide or show the columns at the specified index. 
        //(Note: This seems to be the only working way to hide/show items from a gridview databound from a database.
        protected void showHide(int index, bool boolVal)
        {
            try
            {
                // Remove the header at the specified index.
                gvDept.HeaderRow.Cells[index].Visible = boolVal;
                // Remove each cell of each row at the specified index (the entire column at the specified index).
                foreach (GridViewRow gvr in gvDept.Rows)
                {
                    gvr.Cells[index].Visible = boolVal;
                }
            }
            catch (Exception)
            {
                return;
            }
            
        }
        // Determine if the user selected all depts, or just a specific dept.
        public DataSet determineOptionalParam(int deptNum)
        {
            DataSet ds = new DataSet();
            UserBL us = new UserBL();

            // If deptNum is 1, then the user wants all depts. Pass an optional param in.
            if (deptNum == 1)
            {
                ds = us.populateGridview(deptNum, true);
            }
            // Else the result set will be filtered by a specific deptID.
            else
            {
                ds = us.populateGridview(deptNum);
            }
            return ds;
        }


  
    } // end class
}
