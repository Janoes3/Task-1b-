FUNCTION AddWorkout(userID)
BEGIN FUNCTION
    DECLARE workoutType AS STRING
    DECLARE workoutDate
    DECLARE workoutDurationMins
    DECLARE workoutIntensity
    DECLARE calories
    DECLARE workoutID
    DECLARE isValid
    DECLARE errorMessage AS STRING

    SET isValid TO TRUE
    SET errorMessage TO ""

    # --- Get inputs ---
    SEND "Enter workout type (e.g., Cardio, Strength, Yoga):" TO DISPLAY
    RECEIVE workoutType FROM KEYBOARD

    SEND "Enter workout date and time (DD/MM/YYYY HH:MM):" TO DISPLAY
    RECEIVE workoutDate FROM KEYBOARD

    SEND "Enter workout duration in minutes (1-300):" TO DISPLAY
    RECEIVE workoutDurationMins FROM KEYBOARD

    SEND "Enter workout intensity (Low/Medium/High):" TO DISPLAY
    RECEIVE workoutIntensity FROM KEYBOARD

    SEND "Enter calories burned (0-10000):" TO DISPLAY
    RECEIVE calories FROM KEYBOARD
    # -------------------------
    #   PRESENCE CHECKS
    # -------------------------
    IF workoutType = "" THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Workout type empty")
    END IF

    IF isValid = TRUE AND workoutDate = "" THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Workout date missing")
    END IF

    IF isValid = TRUE AND workoutDurationMins = "" THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Invalid workout duration")
    END IF

    IF isValid = TRUE AND workoutIntensity = "" THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Invalid intensity")
    END IF

    IF isValid = TRUE AND calories = "" THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Invalid calories")
    END IF

    # -------------------------
    #   TYPE / RANGE VALIDATION
    # -------------------------

    # Workout type must be alphabetic
    IF isValid = TRUE AND NOT (workoutType CONTAINS ONLY LETTERS) THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Workout type must contain only letters.")
    END IF

    # Workout date cannot be in the future
    IF isValid = TRUE AND workoutDate > CURRENT_TIME THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Workout future date")
    END IF

    # Duration range
    IF isValid = TRUE AND (workoutDurationMins < 1 OR workoutDurationMins > 300) THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Invalid workout duration")
    END IF

    # Intensity check
    SET workoutIntensity TO UPPER(workoutIntensity)
    IF isValid = TRUE AND NOT (workoutIntensity = "LOW" OR workoutIntensity = "MEDIUM" OR workoutIntensity = "HIGH") THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Invalid intensity")
    END IF

    # Calories range
    IF isValid = TRUE AND (calories < 0 OR calories > 10000) THEN
        SET isValid TO FALSE
        CALL ShowErrorMessage("Invalid calories")
    END IF

    # -------------------------
    #   VALIDATION FAILED
    # -------------------------
    IF isValid = FALSE THEN
        WRITE Tbl_ErrorLog "Workout validation failed", CURRENT_TIME
        WRITE Tbl_WorkoutAudit 0, userID, CURRENT_TIME, "CreateFailed"
        RETURN 0
    END IF
    # -------------------------
    #   INSERT RECORD
    # -------------------------
    SET workoutID TO INSERT INTO Tbl_Workout
        VALUES (AUTO_ID,
                userID,
                workoutType,
                workoutDate,
                workoutDurationMins,
                workoutIntensity,
                calories,
                CURRENT_TIME)

    WRITE Tbl_WorkoutAudit workoutID, userID, CURRENT_TIME, "Created"

    SEND "Workout recorded successfully. ID: " & workoutID TO DISPLAY

    RETURN workoutID
END FUNCTION

















FUNCTION ShowErrorMessage(errorType)
BEGIN FUNCTION
    
    # --- LOGIN ERRORS ---
    IF errorType = "Missing username/email" THEN
        SEND "Enter your username/email." TO DISPLAY

    ELSE IF errorType = "Missing password" THEN
        SEND "Enter your password." TO DISPLAY

    ELSE IF errorType = "Invalid login" THEN
        SEND "Login failed. Check your login details and try again." TO DISPLAY

    ELSE IF errorType = "Account locked" THEN
        SEND "Your account is temporarily locked due to too many failed attempts." TO DISPLAY

    # --- REGISTRATION / CREATE USER ERRORS ---
    ELSE IF errorType = "Invalid email format" THEN
        SEND "Enter a valid email address (example@mail.com)." TO DISPLAY

    ELSE IF errorType = "User already exists" THEN
        SEND "An account with this email already exists." TO DISPLAY

    ELSE IF errorType = "Missing personal details" THEN
        SEND "All personal details must be provided." TO DISPLAY


    # --- DASHBOARD ERRORS ---
    ELSE IF errorType = "Invalid option" THEN
        SEND "Please select a valid option from the dashboard menu." TO DISPLAY

    # --- WORKOUT ERRORS ---
    ELSE IF errorType = "Workout type empty" THEN
        SEND "Workout type cannot be empty." TO DISPLAY

    ELSE IF errorType = "Workout date missing" THEN
        SEND "Workout date/time is required." TO DISPLAY

    ELSE IF errorType = "Workout future date" THEN
        SEND "Workout date/time cannot be in the future." TO DISPLAY

    ELSE IF errorType = "Invalid intensity" THEN
        SEND "Intensity must be Low, Medium, or High." TO DISPLAY

    ELSE IF errorType = "Invalid workout duration" THEN
        SEND "Duration must be between 1 and 300 minutes." TO DISPLAY

    ELSE IF errorType = "Invalid calories" THEN
        SEND "Calories must be between 0 and 10000." TO DISPLAY

    # --- PASSWORD UPDATE ERRORS ---
    ELSE IF errorType = "Password too short" THEN
        SEND "New password must be at least 8 characters long." TO DISPLAY

    ELSE IF errorType = "Password mismatch" THEN
        SEND "Passwords do not match. Try again." TO DISPLAY

    ELSE IF errorType = "Weak password" THEN
        SEND "Password must contain upper, lower, number, and symbol." TO DISPLAY

    ELSE IF errorType = "Password reuse" THEN
        SEND "You cannot reuse a previous password." TO DISPLAY

    ELSE IF errorType = "Same password" THEN
        SEND "New password cannot be the same as your current password." TO DISPLAY

    # --- MEAL PLAN ERRORS ---
    ELSE IF errorType = "Meal day missing" THEN
        SEND "Please enter the day you want to check your meal plan for." TO DISPLAY

    ELSE IF errorType = "No meals found" THEN
        SEND "No meal plan exists for the selected day." TO DISPLAY

    # --- GENERIC / FALLBACK ERRORS ---
    ELSE IF errorType = "Invalid input" THEN
        SEND "The input you entered is not valid. Please try again." TO DISPLAY
    ELSE
        SEND "An error occurred. Please contact support if the problem continues." TO DISPLAY
    END IF

    # Log all errors
    WRITE Tbl_ErrorLog errorType, CURRENT_TIME

END FUNCTION

FUNCTION CreateUser()
BEGIN FUNCTION
    DECLARE userEmail
    DECLARE userPassword
    DECLARE clientName
    DECLARE clientSurname
    DECLARE clientAddress
    DECLARE passwordHash
    DECLARE userID

    # --- Gather required user details ---
    SEND "Enter your email:" TO DISPLAY
    RECEIVE userEmail FROM (ALPHANUMERIC) KEYBOARD

    SEND "Enter your password:" TO DISPLAY
    RECEIVE userPassword FROM (ALPHANUMERIC) KEYBOARD

    SEND "Enter your first name:" TO DISPLAY
    RECEIVE clientName FROM (ALPHABETIC) KEYBOARD

    SEND "Enter your surname:" TO DISPLAY
    RECEIVE clientSurname FROM (ALPHABETIC) KEYBOARD

    SEND "Enter your address:" TO DISPLAY
    RECEIVE clientAddress FROM (ALPHANUMERIC) KEYBOARD


    # --- Validate presence of required inputs ---
    IF userEmail = "" THEN
        CALL ShowErrorMessage("Missing username/email")
        WRITE Tbl_ErrorLog "Missing email", CURRENT_TIME
        RETURN 0
    END IF

    IF userPassword = "" THEN
        CALL ShowErrorMessage("Missing password")
        WRITE Tbl_ErrorLog "Missing password", CURRENT_TIME
        RETURN 0
    END IF

    IF clientName = "" OR clientSurname = "" OR clientAddress = "" THEN
        CALL ShowErrorMessage("Invalid input")
        WRITE Tbl_ErrorLog "Missing personal details", CURRENT_TIME
        RETURN 0
    END IF


    # --- Validate email format (same style as ValidateEntry) ---
    IF NOT (userEmail CONTAINS "@" AND userEmail CONTAINS ".") THEN
        CALL ShowErrorMessage("Invalid input")
        WRITE Tbl_ErrorLog "Invalid email format", CURRENT_TIME
        RETURN 0
    END IF


    # --- Check if user already exists ---
    SET userID TO SELECT user_id FROM Tbl_User WHERE email = userEmail
    IF userID <> 0 THEN
        CALL ShowErrorMessage("Invalid login")
        WRITE Tbl_ErrorLog "Account already exists", CURRENT_TIME
        RETURN 0
    END IF


    # --- Hash the password (same method used in ValidateLoginCredentials) ---
    SET passwordHash TO HashPassword(userPassword)


    # --- Insert the new user record ---
    SET userID TO INSERT INTO Tbl_User
                VALUES (AUTO_ID,
                        userEmail,
                        passwordHash,
                        clientName,
                        clientSurname,
                        clientAddress,
                        CURRENT_TIME)

    # --- Log user creation ---
    WRITE Tbl_UserAudit userID, CURRENT_TIME, "Created"

    SEND "Account created successfully." TO DISPLAY

    RETURN userID
END FUNCTION




FUNCTION CreateClientDashboard(usernameEmail, userID)
BEGIN FUNCTION
    DECLARE sessionActive
    DECLARE choice
    DECLARE invalidAttempts
    DECLARE recentWorkouts
    DECLARE upcomingWorkshops
    DECLARE todaysMeals

    SET sessionActive TO TRUE
    SET invalidAttempts TO 0

    # --- Audit entry for opening dashboard ---
    WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "Open"

    # --- Optional: show a short summary panel on load ---
    SET recentWorkouts TO SELECT TOP 3 workout_type, workout_date, workout_duration_mins
                         FROM Tbl_Workout
                         WHERE user_id = userID
                         ORDER BY workout_date DESC

    SET upcomingWorkshops TO SELECT TOP 3 workshop_name, workshop_date, workshop_format
                             FROM Tbl_WorkshopBooking
                             WHERE user_id = userID AND workshop_date >= CURRENT_DATE
                             ORDER BY workshop_date ASC

    SET todaysMeals TO SELECT meal_type, calories
                       FROM Tbl_MealPlan
                       WHERE user_id = userID AND meal_day = DAY_OF_WEEK(CURRENT_DATE)

    SEND "Dashboard loaded" TO DISPLAY
    SEND "Welcome, " & usernameEmail & "!" TO DISPLAY

    # --- Summary cards (lightweight, non-blocking) ---
    SEND "Recent workouts:" TO DISPLAY
    FOR EACH w FROM recentWorkouts DO
        SEND "- " & w.workout_type & " on " & FORMAT(w.workout_date, "DD/MM/YYYY HH:MM") &
             " for " & w.workout_duration_mins & " mins" TO DISPLAY
    END FOREACH

    SEND "Today's meals:" TO DISPLAY
    FOR EACH m FROM todaysMeals DO
        SEND "- " & m.meal_type & " (" & m.calories & " kcal)" TO DISPLAY
    END FOREACH

    SEND "Upcoming workshops:" TO DISPLAY
    FOR EACH ws FROM upcomingWorkshops DO
        SEND "- " & ws.workshop_name & " on " & FORMAT(ws.workshop_date, "DD/MM/YYYY HH:MM") &
             " [" & ws.workshop_format & "]" TO DISPLAY
    END FOREACH

    # --- Main dashboard loop ---
    WHILE sessionActive = TRUE DO
        SEND "" TO DISPLAY
        SEND "Options:" TO DISPLAY
        SEND "1. Log your workout" TO DISPLAY
        SEND "2. Check meal plan" TO DISPLAY
        SEND "3. Watch a session" TO DISPLAY
        SEND "4. Check user progress" TO DISPLAY
        SEND "5. Exit" TO DISPLAY

        SEND "Enter your choice (1-5):" TO DISPLAY
        RECEIVE choice FROM (INTEGER) KEYBOARD

        # --- Validate menu choice ---
        IF choice < 1 OR choice > 5 THEN
            SET invalidAttempts TO invalidAttempts + 1
            CALL ShowErrorMessage("Invalid option")
            WRITE Tbl_ErrorLog "Invalid dashboard choice: " & choice, CURRENT_TIME
            WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "InvalidChoice"
            IF invalidAttempts >= 3 THEN
                SEND "Too many invalid attempts. Returning to login." TO DISPLAY
                WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "ExitTooManyInvalidChoices"
                SET sessionActive TO FALSE
            END IF
        ELSE
            # --- Reset invalid attempt counter on valid input ---
            SET invalidAttempts TO 0

            # --- Route to selected feature ---
            IF choice = 1 THEN
                WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "Select_LogWorkout"
                CALL AddWorkout(userID)
                # Optionally refresh summaries after action
                GOTO RefreshSummary

            ELSE IF choice = 2 THEN
                WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "Select_CheckMealPlan"
                CALL ViewMealPlan(userID)  # Assumes a function to view meal plan details
                GOTO RefreshSummary

            ELSE IF choice = 3 THEN
                WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "Select_WatchSession"
                CALL WatchSession(userID)  # Assumes a function to browse/stream sessions
                GOTO RefreshSummary

            ELSE IF choice = 4 THEN
                WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "Select_CheckUserProgress"
                CALL ViewUserProgress(userID)  # Assumes a function to view progress charts
                GOTO RefreshSummary

            ELSE IF choice = 5 THEN
                WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "Select_Exit"
                SET sessionActive TO FALSE
            END IF
        END IF

        CONTINUE  # back to top of WHILE

        # --- Lightweight dashboard refresh after actions ---
        LABEL RefreshSummary
            SET recentWorkouts TO SELECT TOP 3 workout_type, workout_date, workout_duration_mins
                                 FROM Tbl_Workout
                                 WHERE user_id = userID
                                 ORDER BY workout_date DESC

            SET todaysMeals TO SELECT meal_type, calories
                               FROM Tbl_MealPlan
                               WHERE user_id = userID AND meal_day = DAY_OF_WEEK(CURRENT_DATE)

            SEND "" TO DISPLAY
            SEND "Quick summary updated:" TO DISPLAY
            SEND "Recent workouts:" TO DISPLAY
            FOR EACH w2 FROM recentWorkouts DO
                SEND "- " & w2.workout_type & " on " & FORMAT(w2.workout_date, "DD/MM/YYYY HH:MM") &
                     " for " & w2.workout_duration_mins & " mins" TO DISPLAY
            END FOREACH

            SEND "Today's meals:" TO DISPLAY
            FOR EACH m2 FROM todaysMeals DO
                SEND "- " & m2.meal_type & " (" & m2.calories & " kcal)" TO DISPLAY
            END FOREACH
        # end RefreshSummary

    END WHILE

    # --- Exit and tidy up ---
    SEND "Goodbye!" TO DISPLAY
    WRITE Tbl_DashboardAudit userID, CURRENT_TIME, "Close"

    RETURN TRUE
END FUNCTION