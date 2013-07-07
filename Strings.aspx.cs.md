Test_Repo
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;

namespace CSharpApp
{
    public partial class Strings : System.Web.UI.Page
    {
        protected void Page_Load(object sender, EventArgs e)
        {
            if (!IsPostBack)
            {
                lblResult.Visible = false;
            }
            if (IsPostBack)
            {
                if (String.IsNullOrEmpty(txtInput.Text) || String.IsNullOrWhiteSpace(txtInput.Text))
                {
                    lblResult.Text = "The input string is empty.";
                    lblResult.Visible = true;
                   
                }
            }
           
        } // end Page_Load

        // Method to reverse all chars in a string.
        protected void btnReverseChars_Click(object sender, EventArgs e)
        {
            // Get the string input by the user.
            string newString = txtInput.Text;

            // Break the string into an array of chars.
            char[] inOrder = newString.ToCharArray();

            // Create a string to be used later.
            string finalString = "";

            // Grab each character starting from the last index in char array...
            for (int i = inOrder.Length - 1; i >= 0; i--)
            {
                // Grab the character at i and concatenads1kin ;lm2te it to 'finalString'.
                // This is the reversal process.
                finalString += inOrder[i].ToString();
            } // end for loop

            // Output the string to our output label.
            lblResult.Text = finalString;
            lblResult.Visible = true;
        } // end btnReverseChars_Click()

        protected void btnReverseString_Click(object sender, EventArgs e)
        {
            // Get the string input by the user.
            string newString = txtInput.Text;
            // Create a string for later use .
            string finalString = "";
            // Split the string into a string array. Split on space.
            string[] stringArray = newString.Split(' ');
            /* Start at the last index of the array. Grab each word and put concatenate it to our string. 
             * Also, add a space between each word since it was removed when we split the string into an array.
             */
            for (int i = stringArray.Length - 1; i >= 0; i--)
            {
                finalString += stringArray[i] + ' ';
            }

            // Output the string to our label.
            lblResult.Text = finalString;
            lblResult.Visible = true;
        }

        // Remove whitespaces from the input string.
        protected void btnRemoveWhitespaces_Click(object sender, EventArgs e)
        {
            // Create an output string for later use.
            string finalString = "";
            // Split the input string on whitespaces.
            string[] stringArray = txtInput.Text.Split(' ');
            // Take each string in the array and concat it to a string variable.
            for (int i = 0; i < stringArray.Length; i++)
            {
                finalString += stringArray[i];
            }

            // Output the string to output label.
            lblResult.Text = finalString;
            lblResult.Visible = true;
        } // end btnReverseString_Click()

        protected void btnCountOccurences_Click(object sender, EventArgs e)
        {
            // If the user doesn't enter any characters, then give them an error message.
            if(String.IsNullOrEmpty(txtCountChar.Text) || String.IsNullOrWhiteSpace(txtCountChar.Text))
            {
                lblResult.Text = "The search string is empty.";
                lblResult.Visible = true;
                return;
            }
            // Create a dictionary for later use. 
            Dictionary<string, int> dict = new Dictionary<string, int>();
            //Dictionary<string, int> dict = new Dictionary<string, int> { { "a", 0 }, { "b", 0 }, { "c", 0 }, { "d", 0 }, { "e", 0 }, 
            //{"f",0},{"g",0},{"h",0},{"i",0},{"j",0},{"k",0},{"l",0},{"m",0},{"n",0},{"o",0},{"p",0},{"q",0},{"r",0},{"s",0}, {"t",0},
            //{"u",0},{"v",0},{"w",0},{"x",0},{"y",0},{"z",0} };

            // Get the string that we are checking for.
            string checkKey = txtCountChar.Text;

            // Get the input string that was typed by the user.
            string inputString = txtInput.Text;

            // Populate the dictionary. This will take the input string and store the number of occurences of each char.
            dict = populateDictionary(inputString, dict);

            // Check if the dictionary contains the char that we are searching for.
            int value = 0;
            dict.TryGetValue(checkKey, out value);

            // If the dictionary contains the value we are looking for, then output its number of occurences.
            if (value > 0)
            {
                lblResult.Text = "The occurences of '" + checkKey + "' in this string is: " + value.ToString();
                lblResult.Visible = true;
            }
            // Else output that there are no occurences of that character.
            else
            {
                lblResult.Text = "There are no occurences of '" + checkKey + "' in this string.";
                lblResult.Visible = true;
            }

        } // end btnCountOccurences_Click()

        // Checks the input string for the chars to be transposed and swaps them, if found.
        protected void btnChange_Click(object sender, EventArgs e)
        {
            lblResult.Text = "";
            string[] transposeArray = null;
            //Get each of the characters we want to change.
            try
            {
                transposeArray = txtTransChars.Text.Split(',');
            }
            catch(Exception ex)
            {
                lblResult.Text = "Your entry must be two characters separated by a comma.";
                lblResult.Visible = true;
                return;
            }
                

            // If the search string is too small, then output an error message to the user.
            if (transposeArray.Length < 2)
            {
                lblResult.Text = "Your entry must be two characters separated by a comma.";
                lblResult.Visible = true;
                return;
            }
        
            // An output string for later use.
            string outputString = "";

            // Split the input string into a char Array.
            char[] charsArray = txtInput.Text.ToCharArray();

            // Get both of the strings in the search string.
            string _1stChar = transposeArray[0];
            string _2ndChar = transposeArray[1];

            // Make sure the string contains both of the search strings.
            if (!txtInput.Text.Contains(_1stChar))
            {
                lblResult.Text = "The input string does not contain '" + _1stChar + " '";
                lblResult.Visible = true;
                return;
            }
            else if (!txtInput.Text.Contains(_2ndChar))
            {
                lblResult.Text = "The input string does not contain '" + _2ndChar + " '";
                lblResult.Visible = true;
                return;
            }
            
            // Go through the char Array (the input string) and check for either of the specified chars.
            for (int i = 0; i <= charsArray.Length - 1; i++)
            {
                // If the first search char is found in the string, swap it with the character at transposeArray[1].
                if (charsArray[i].ToString() == _1stChar)
                {
                    outputString += _2ndChar;
                }
                // Else If the second search char is found in the string, swap it with the character at transposeArray[0]
                else if (charsArray[i].ToString() == _2ndChar)
                {
                    outputString += _1stChar;
                }
                // Else just concat it to the output string.
                else
                {
                    outputString += charsArray[i];
                }

            } // end for loop   

            // Output the string
            lblResult.Text = outputString;
            lblResult.Visible = true;
        } // btnChange_Click()

        public Dictionary<string, int> populateDictionary(string inputString, Dictionary<string, int> dict)
        {
            // Split the input string into a char array.
            char[] charArray = inputString.ToCharArray();
            // Create a value to hold the number of occurences of a char.
            int value = 0;
            // Iterate through the chars in the input string and store number of their occurences in the dictionary.
            for (int i = 0; i < charArray.Length; i++)
            {
                // Change each char into a string, since the dictionary has strings as keys.
                string checkKey = charArray[i].ToString();
                // If the string is already in the dictionary, increment its number of occurences (its value).
                if (dict.ContainsKey(checkKey))
                {
                    // Get its value
                    dict.TryGetValue(checkKey, out value);
                    // Increment value.
                    value++;
                    // Update the value of checkKey in the dictionary with the new incremented value.
                    dict[checkKey] = value;
                } // end if

                // Else the key wasn't found in the dictionary. Add this key and default its value to 1.
                else
                {
                    dict.Add(checkKey, 1);
                } // end else

            } // end for loop
            return dict;
        } // end populateDictionary()

        protected void btnFindHighest_Click(object sender, EventArgs e)
        {
      
            // Split the input string into a character array.
            char[] charArray = txtInput.Text.ToCharArray();

            // Create a highest string (occurs latest in the alphabet).
            string strHigh = null;

            for(int i = 0; i < charArray.Length; i++ )
            {
                if (strHigh == null)
                {
                    strHigh = charArray[i].ToString();
                } // end if
                else
                {
                    // Compare the char at our current index with the current high. 
                    // If it is higher (returns 1), then set it as the new high.
                    if (charArray[i].ToString().CompareTo(strHigh) == 1 && !charArray[i].ToString().Equals(" "))
                    {
                        strHigh = charArray[i].ToString();
                    } // end if
                } // end else
            } // end for loop
            if (!String.IsNullOrEmpty(strHigh))
            {
                lblResult.Text = "The highest character in the input string is '" + strHigh + "'.";
                lblResult.Visible = true;
            }
            

        } // end btnFindHighest_Click()

        protected void btnFindLowest_Click(object sender, EventArgs e)
        {
            // Split the input string into a character array.
            char[] charArray = txtInput.Text.ToCharArray();

            // Create a lowest string (occurs earliest in the alphabet).
            string strLow = null;

            for (int i = 0; i < charArray.Length; i++)
            {
                // If strLow is null then it hasn't yet been assigned a value. Assign it the value of the first character we encountered.
                if (strLow == null)
                {
                    strLow = charArray[i].ToString();
                } // end if
                else
                {
                    // Compare the char at our current index with the current high. 
                    // If it is higher (returns 1), then set it as the new high.
                    if (charArray[i].ToString().CompareTo(strLow) == -1 && !charArray[i].ToString().Equals(" "))
                    {
                        strLow = charArray[i].ToString();
                    } // end if
                } // end else
            } // end for loop

            if (!String.IsNullOrEmpty(strLow))
            {
                lblResult.Text = "The highest character in the input string is '" + strLow + "'.";
                lblResult.Visible = true;
            }
        }

        protected void btnTranspose_Click(object sender, EventArgs e)
        {
            // Make sure the search string is not empty.
            if (String.IsNullOrEmpty(txtTranspose.Text))
            {
                lblResult.Text = "The search string is empty.";
                lblResult.Visible = true;
                return;
            }
            // Split the input string into a char array.
            char[] charArray = txtInput.Text.ToCharArray();

            // Create a string array for later use.
            string[] searchArray = null;

            // Create an output string for later use.
            string outputString = "";

            // Try to split the search string on comma. If it works, then the user entered values correctly.
            try
            {
                searchArray = txtTranspose.Text.Split(',');
            }
            catch (Exception)
            {
                lblResult.Text = "The search string must be two characters, separated by a comma.";
                lblResult.Visible = true;
                return;
            }
            // Ensure that the search string has two comma delimited chars
            if (searchArray.Length < 2)
            {
                lblResult.Text = "The search string must be two characters, separated by a comma.";
                lblResult.Visible = true;
                return;
            }
       
            /* Iterate through the input string. If any of our search characters occur next to each other,
               then swap their positions. */
            for (int i = 0; i < charArray.Length-1; i++)
            {
                // If our search string is a,b and we have "ab", then...
                if (charArray[i].ToString().Equals(searchArray[0]) && charArray[i + 1].ToString().Equals(searchArray[1]))
                {
                    // Swap the chars.
                    char temp = charArray[i];
                    charArray[i] = searchArray[1][0];
                    charArray[i + 1] = temp;
                } // end if
                // Else if our search string is a,b and we have "ba", then...
                else if (charArray[i].ToString().Equals(searchArray[1]) && charArray[i + 1].ToString().Equals(searchArray[0]))
                {
                    // Swap the chars.
                    char temp = charArray[i];
                    charArray[i] = searchArray[0][0];
                    charArray[i + 1] = temp;
                } // end if
                
            } // end for loop

            // Iterate through the char array and output the resultant string.
            for (int i = 0; i < charArray.Length; i++)
            {
                outputString += charArray[i];
            }

            lblResult.Text = "The new string is: " + outputString;
            lblResult.Visible = true;
        }

        protected void btnRemove_Click(object sender, EventArgs e)
        {
            if (String.IsNullOrEmpty(txtRemove.Text))
            {
                lblResult.Text = "The search string is empty.";
                lblResult.Visible = true;
                return;
            }
            // Split the input string into a char array
            char[] charArray = txtInput.Text.ToCharArray();
            // Get the first character of the search string.
            char c = txtRemove.Text[0];

            // Delete all occurences of the search character from the array.
            charArray = deleteAllFromArray(charArray, c);

            // Create a string to output to user.
            string outputString = "";

            // Iterate through the array and append each of its chars to the string.
            for (int i = 0; i < charArray.Length; i++)
            {
                outputString += charArray[i].ToString();
            }
            lblResult.Text = outputString;
            lblResult.Visible = true;
        } // end method()

        // Delete from a char array.
        public char[] deleteAllFromArray(char[] array, char c)
        {
            // Create a List so that we an have a dynamic container to add elements into.
            List<char> cList = new List<char>();

            for (int j = 0; j < array.Length; j++)
            {
                // If the char at j is the same as the search value, then skip this element.
                if (array[j].Equals(c))
                {
                    continue;
                }
                // Else, it is a valid char. Copy it into the List
                else
                {
                    cList.Add(array[j]);

                }
            } // end for loop

            // Create a new char array.
            char[] resizedArray = new char[cList.Count];

            // Copy all of the elements from the list into the char array.
            for (int i = 0; i < cList.Count; i++)
            {
                resizedArray[i] = cList[i];
            }
            // Return the char array.
            return resizedArray;
        }

      
    } // end _Default
} // end nameSpace CSharpApp
