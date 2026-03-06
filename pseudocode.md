FUNCTION ShowErrorMessage(errorType)
BEGIN FUNCTION
    IF errorType = "Missing username/email" THEN
        SEND "Enter your username/email" TO DISPLAY
    ELSE IF errorType = "Missing password" THEN
        SEND "Enter your password" TO DISPLAY
    ELSE IF errorType = "Invalid login" THEN
        SEND "Login failed. Check your details and try again." TO DISPLAY
    ELSE
        SEND "An error occurred. Please contact support if the problem continues." TO DISPLAY
    END IF
 # Log the error for audit and support
 WRITE Tbl_ErrorLog errorType, CURRENT_TIME

END FUNCTION 


FUNCTION AddOrRetrieveClientRecord(clientFirstName, clientSurname, clientAddress, 
clientPostcode)
BEGIN FUNCTION
    DECLARE clientID
    DECLARE clientExists
    SET clientExists TO FALSE
    SET clientID TO 0
    
    # Lookup client in tbl_client by surname and postcode
    SET clientID TO SELECT client_id FROM tbl_client WHERE surname = clientSurname AND 
    postcode = clientPostcode
    IF clientID <> 0 THEN
        SET clientExists TO TRUE
    END IF

    IF clientExists = TRUE THEN
    # Log retrieval for audit
        WRITE Tbl_ClientAudit clientID, CURRENT_TIME, "Retrieved"
        RETURN clientID
    ELSE
    Task 1B Exemplar Scenario: CateringPro
    11
    # Insert new client record into tbl_client
        SET clientID TO INSERT INTO tbl_client VALUES (clientFirstName, clientSurname, 
    clientAddress, clientPostcode)
    # Log creation for audit
        WRITE Tbl_ClientAudit clientID, CURRENT_TIME, "Created"
        RETURN clientID
    END IF
END FUNCTION



FUNCTION ValidateEntry(inputValue)
BEGIN FUNCTION

    DECLARE isValid
    DECLARE errorMessage

    SET isValid TO TRUE
    SET errorMessage TO ""

    # Presence check
    IF inputValue = "" THEN
        SET isValid TO FALSE
        SET errorMessage TO "Input cannot be empty."
    END IF

    # Format check (example: email format)
    IF isValid = TRUE THEN
        IF NOT (inputValue CONTAINS "@" AND inputValue CONTAINS ".") THEN
            SET isValid TO FALSE
            SET errorMessage TO "Input must be a valid email address."
        END IF
    END IF

    # Type check (example: input should be INTEGER)
    IF isValid = TRUE THEN
        IF NOT (inputValue = inputValue DIV 1) THEN
            SET isValid TO FALSE
            SET errorMessage TO "Input must be a whole number."
        END IF
    END IF

    # Range check (example: input between 1 and 100)

    IF isValid = TRUE THEN
        IF inputValue < 1 OR inputValue > 100 THEN
            SET isValid TO FALSE
            SET errorMessage TO "Input must be between 1 and 100."
        END IF
    END IF

    IF isValid = TRUE THEN
        RETURN "Valid"
    ELSE
        RETURN errorMessage
    END IF
END FUNCTION


FUNCTION ValidateLoginCredentials(usernameEmail, password)
BEGIN FUNCTION

    SET loginValid TO FALSE

    # Retrieve password hash for the given email from Tbl_User
    SET storedPasswordHash TO SELECT password_hash FROM Tbl_User WHERE email = 
usernameEmail

    # Log the login attempt for audit purposes
    WRITE Tbl_LoginAudit usernameEmail, CURRENT_TIME, "Attempt"

    IF storedPasswordHash <> "" THEN
        IF VerifyPassword(password, storedPasswordHash) = TRUE THEN
            SET loginValid TO TRUE
    # Log successful login
        WRITE Tbl_LoginAudit usernameEmail, CURRENT_TIME, "Success"
        ELSE
        # Log failed password verification
            WRITE Tbl_LoginAudit usernameEmail, CURRENT_TIME, "FailedPassword"
        END IF
    ELSE
    # Log failed email lookup
        WRITE Tbl_LoginAudit usernameEmail, CURRENT_TIME, "FailedEmail"
    END IF
        
    RETURN loginValid
END FUNCTION