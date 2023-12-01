get_club_form Shortcode:

This shortcode generates a display of a football club's recent form based on the outcomes of its latest matches.
The shortcode accepts parameters such as club_id, date_to, and classes.
It uses the anwp_football_leagues functions to retrieve and organize match data, and then it displays the outcomes with associated labels and styles.
custom_prediction_shortcode Shortcode:

This shortcode is designed to display custom predictions for football matches.
It accepts attributes, extracts data from post meta, and calculates average values for home and away teams.
The calculated values are used to create a visual representation of the predictions with colored bars.
Additional Information:

Both shortcodes use output buffering (ob_start and ob_get_clean) to capture the HTML output.
The styles and classes used in the HTML output seem to be designed for football-related elements, such as outcome labels and prediction bars.
Overall, these functions and shortcodes appear to be part of a larger system or plugin related to [football predictions](https://escored.com) and statistics within a WordPress environment. If you have specific questions or if you need further clarification on any part of the code, feel free to ask!
