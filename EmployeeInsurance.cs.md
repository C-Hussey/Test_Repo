using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using CSharpApp.BusinessLayer;
using CSharpApp.UtilityObj;
using System.Data;

namespace CSharpApp.Pages
{   
    public partial class EmployeeInsurance : System.Web.UI.Page
    {

        protected void Page_Load(object sender, EventArgs e)
        {
            /* The following controls need registering in order to cause postback to an update panel 
             * from within another update panel. */
            ScriptManager.GetCurrent(this.Page).RegisterPostBackControl(btnAddEmpInsurance);
            ScriptManager.GetCurrent(this.Page).RegisterPostBackControl(btnAddDepInsurance);
            ScriptManager.GetCurrent(this.Page).RegisterPostBackControl(btnFinalAddEmpIns);
            ScriptManager.GetCurrent(this.Page).RegisterPostBackControl(btnFinalAddDepIns);
     
            // Bind the headers to the appropriate gridviews for display purposes.
            EmployeeBL em = new EmployeeBL();
            gvDependants.DataSource = em.selectHeaders(gvDependants.ID);
            gvDependants.DataBind();
            gvEmployees.DataSource = em.selectHeaders(gvEmployees.ID);
            gvEmployees.DataBind();
            gvInsuranceHistory.DataSource = em.selectHeaders(gvInsuranceHistory.ID);
            gvInsuranceHistory.DataBind();

            // Reset any controls (labels, textboxes, etc) that may need to be reset to default values.
            resetFields();

            
            if (!IsPostBack)
            {            
                // Get the data for the employee info dropdown list.   
                List<Employee> employeeList = new List<Employee>();
                employeeList = em.selectAllEmployees();
                // Add a default selection at index 0.
                ddlSelectEmployee.Items.Add(new ListItem("--Select--", "0"));

                // Populate the dropdown list with employee objects.
                foreach (Employee employee in employeeList)
                {
                    ddlSelectEmployee.Items.Add(new ListItem(employee.EmployeeName,employee.EmployeeID.ToString()));
                }

                // Populate the employee insurance dropdown list
                List<string> empInsuranceTypes = em.getEmployeeInsuranceTypes();
                for (int i = 0; i < empInsuranceTypes.Count; i++)
                {
                    ddlEmpInsurance.Items.Add(new ListItem(empInsuranceTypes[i], empInsuranceTypes[i]));
                }

                // Populate the dependant insurance dropdown list
                List<string> depInsuranceTypes = em.getDepInsuranceTypes();
                for (int i = 0; i < depInsuranceTypes.Count; i++)
                {
                    ddlDepInsurance.Items.Add(new ListItem(depInsuranceTypes[i], depInsuranceTypes[i]));
                }
            }
            
            // If an employee was actually selected from the employee dropdown list...
            if (ddlSelectEmployee.SelectedIndex > 0)
            {
                int employeeID = Convert.ToInt32(ddlSelectEmployee.SelectedValue);
                try
                {
                    // Update/show the employee and insurance history gridviews, based on this employee.
                    gvEmployees.DataSource = em.getEmployeeInfoByEmpID(employeeID);
                    gvEmployees.DataBind();               
                    gvInsuranceHistory.DataSource =  em.getCancelledInsurance(employeeID);
                    gvInsuranceHistory.DataBind();
                    // Enable the button so that the user can add new insurance types
                    btnAddEmpInsurance.Enabled = true;
                }
                catch (Exception ex)
                {

                }
                /* If the employee has dependants, then populate the dependant gridview. 
                 * Otherwise, notify the user that there are no dependants for this employee.*/
                if (em.employeeHasDependants(employeeID))
                {
                    gvDependants.DataSource = em.getDependantsInfoByEmpID(employeeID);
                    btnAddDepInsurance.Enabled = true;
                    gvDependants.DataBind();
                    upnDependants.Update();
                }
                // Else inform the user that this employee has no dependants.
                else
                {
                    lblDepErr.Text = "**No dependant information available for this employee.";
                    gvDependants.DataSource = em.getDependantsInfoByEmpID(employeeID);
                    btnAddDepInsurance.Enabled = false;
                    gvDependants.DataBind();
                    upnDependants.Update();
                }
            }       
        }

        protected void ddlSelectEmployee_SelectedIndexChanged(object sender, EventArgs e)
        {

            EmployeeBL em = new EmployeeBL();
            // Get the selected employee from the dropdownlist.
            int employeeID = Convert.ToInt32(ddlSelectEmployee.SelectedValue);

            // Update/show the gridview, based on this employee.
            gvEmployees.DataSource = em.getEmployeeInfoByEmpID(employeeID);           
            gvEmployees.DataBind();
        }

        // Reset the appropriate fields. Occurs on postback.
        protected void resetFields()
        {
            lblDepErr.Text = "";
            lblInsuranceErr.Text = "";
            lblEmpInsInputErr.Text = "";
            btnAddDepInsurance.Enabled = false;
            btnAddEmpInsurance.Enabled = false;
        }

        // Display a panel to allow the user to add a new Employee Insurance.
        protected void btnAddEmpInsurance_Click(object sender, EventArgs e)
        {
            EmployeeBL em = new EmployeeBL();
            Validation va = new Validation();

            upnAddEmpInsurance.Visible = true;
            upnAddEmpInsurance.Update();
               
        }

        // Display a panel to allow the user to add a new Dependant Insurance.
        protected void btnAddDepInsurance_Click(object sender, EventArgs e)
        {
            EmployeeBL em = new EmployeeBL();
            Validation va = new Validation();

            upnAddDepInsurance.Visible = true;
            upnAddDepInsurance.Update();
        }
        
        // Validate the insurance being added. If validation passes, then add the new dependant insurance.
        protected void btnFinalAddDepIns_Click(object sender, EventArgs e)
        {
            EmployeeBL em = new EmployeeBL();
            Validation va = new Validation();
            // Validate the input as a decimal amount
            if (va.isMoney(txtDepAmount.Text))
            {
                // Get the insurance name from employee dropdown list.
                string insuranceName = ddlDepInsurance.SelectedValue;
                // Get the employee ID from the employee ddl
                int employeeID = ddlSelectEmployee.SelectedIndex;

                // Make sure a duplicate insurance/amount entry does not exist for this employee;s dependant.
                if (!em.depInsuranceExists(employeeID, insuranceName, Convert.ToDecimal(txtDepAmount.Text)))
                {
                    // Add this insurance and amount to the Employee Dependant table
                    em.addNewDepInsurance(employeeID, insuranceName, Convert.ToDecimal(txtDepAmount.Text));
                    // Update the dependants gridview.
                    gvDependants.DataSource = em.getDependantsInfoByEmpID(employeeID);
                    gvDependants.DataBind();
                    upnDependants.Update();
                    // If adding the insurance was successful, hide the update panel which allows for adding the new insurance types.
                    upnAddDepInsurance.Visible = false;
                    upnAddDepInsurance.Update();
                }
                // If a duplicate record was found, notify the user.
                else
                {
                    lblDepInsInputErr.Text = "**Duplicate record";
                    return;
                }

            }
            // Input fails for negative numbers and non-numeric strings.
            else
            {
                lblDepInsInputErr.Text = "**Input is invalid";
            }
        }

        // Validate the insurance being added. If validation passes, then add the new dependant insurance.
        protected void btnFinalAddEmpIns_Click(object sender, EventArgs e)
        {
            EmployeeBL em = new EmployeeBL();
            Validation va = new Validation();
            // Validate the input as a decimal amount
            if (va.isMoney(txtEmplAmount.Text))
            {
                // Get the insurance name from ddl
                string insuranceName = ddlEmpInsurance.SelectedValue;
                // Get the employee ID from the employee ddl
                int employeeID = ddlSelectEmployee.SelectedIndex;

                // Make sure a duplicate insurance/amount entry does not exist for this employee.
                if (!em.employeeInsuranceExists(employeeID, insuranceName, Convert.ToDecimal(txtEmplAmount.Text)))
                {
                    // Add this insurance and amount to the Employee Insurance table
                    em.addNewEmployeeInsurance(employeeID, insuranceName, Convert.ToDecimal(txtEmplAmount.Text));
                    // Update the employee gridview.
                    gvEmployees.DataSource = em.getEmployeeInfoByEmpID(employeeID);
                    gvEmployees.DataBind();
                    upnEmployees.Update();
                    // If adding insurance was successful, hide the update panel which allows for adding the new insurance types.
                    upnAddEmpInsurance.Visible = false;
                    upnAddEmpInsurance.Update();
                }
                // If a duplicate record was found, notify the user.
                else
                {
                    lblEmpInsInputErr.Text = "**Duplicate record";
                    return;
                }

            }
            // Input fails for negative numbers and non-numeric strings
            else
            {
                lblEmpInsInputErr.Text = "**Input is invalid";
            }
        }

    } // end class
}
