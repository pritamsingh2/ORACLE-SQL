# ORACLE-SQL

create sequence trns_seq
start with 1000
increment by 1

create or replace package Conference_Automation_test
IS
    procedure p_insert_booking(p_CCode number, p_HName varchar2,p_StartDate Date, p_EndDate Date);
    procedure p_TotalBooking(p_HType varchar2);
    procedure p_generate_report(p_CCode number);
    procedure p_get_rent(p_HNAME VARCHAR2);
    
end     Conference_Automation_test;
/    





CREATE OR REPLACE PACKAGE BODY Conference_Automation_test
IS
    FUNCTION f_validate_ccode (p_CCode NUMBER)
        RETURN NUMBER
    IS
        v_length_ccode   NUMBER;
    BEGIN
        SELECT LENGTH (p_CCode) INTO v_length_ccode FROM DUAL;

        IF v_length_ccode = 3
        THEN
            IF p_CCode BETWEEN 100 AND 999
            THEN
                RETURN 1;
            ELSE
                RETURN 2;
            END IF;
        ELSE
            RETURN 3;
        END IF;
    END f_validate_ccode;

    FUNCTION f_validate_hall_capacity (p_HType VARCHAR2)
        RETURN NUMBER
    IS
    BEGIN
        IF p_HType IN ('SMALL', 'MEDIUM', 'LARGE')
        THEN
            RETURN 1;
        ELSE
            RETURN 2;
        END IF;
    END f_validate_hall_capacity;


    PROCEDURE p_insert_booking (p_CCode       NUMBER,
                                p_HName       VARCHAR2,
                                p_StartDate   DATE,
                                p_EndDate     DATE)
    IS
        V_CCode         NUMBER;
        V_COUNT_HNAME   NUMBER;
    BEGIN
        V_CCode := f_validate_ccode (p_CCode);

        IF V_CCode = 1
        THEN
            SELECT COUNT (*)
              INTO V_COUNT_HNAME
              FROM HALLS
             WHERE HNAME = p_HName;

            INSERT INTO BOOKING (TRANSID,
                                 CCODE,
                                 HNAME,
                                 STARTDATE,
                                 ENDDATE)
                 VALUES (trns_seq.NEXTVAL,
                         p_CCode,
                         p_HName,
                         p_StartDate,
                         p_EndDate);
        ELSE
            DBMS_OUTPUT.PUT_LINE ('CCODE AND HNAME DOES NOT EXIST');
        END IF;
    EXCEPTION
        WHEN NO_DATA_FOUND
        THEN
            DBMS_OUTPUT.PUT_LINE ('NO DATA FOUND. TRY AGAIN');
        WHEN OTHERS
        THEN
            DBMS_OUTPUT.PUT_LINE ('ERROR OCCURED IS: ' || SQLERRM);
    END p_insert_booking;

    PROCEDURE p_TotalBooking (p_HType VARCHAR2)
    IS
        V_TOTAL_BOOKINGS   NUMBER;
        V_HType            VARCHAR2 (20);
    BEGIN
        V_HType := f_validate_hall_capacity (p_HType);

        IF V_HType = 1
        THEN
            SELECT NOOFBOOKINGS
              INTO V_TOTAL_BOOKINGS
              FROM HALLS
             WHERE HTYPE = p_HType;

            DBMS_OUTPUT.PUT_LINE (
                'TOTAL NO OF BOOKINGS ARE: ' || V_TOTAL_BOOKINGS);
        END IF;
    EXCEPTION
        WHEN NO_DATA_FOUND
        THEN
            DBMS_OUTPUT.PUT_LINE ('NO DATA FOUND. TRY AGAIN');
        WHEN OTHERS
        THEN
            DBMS_OUTPUT.PUT_LINE ('ERROR OCCURED IS: ' || SQLERRM);
    END p_TotalBooking;

    PROCEDURE p_generate_report (p_CCode NUMBER)
    IS
        V_CCode       NUMBER;
        V_TRANSID     NUMBER;
        V_HNAME       VARCHAR2 (100);
        V_STARTDATE   DATE;
        V_ENDDATE     DATE;
    BEGIN
        V_CCode := f_validate_ccode (p_CCode);

        IF V_CCode = 1
        THEN
            FOR X IN (SELECT TRANSID,
                             HNAME,
                             STARTDATE,
                             ENDDATE
                        FROM BOOKING
                       WHERE CCODE = p_CCode)
            LOOP
                DBMS_OUTPUT.PUT_LINE ('THE VALUES ARE');
                DBMS_OUTPUT.PUT_LINE ('TRANSACTION ID IS: ' || X.TRANSID);
                DBMS_OUTPUT.PUT_LINE ('HALL NAME IS: ' || X.HNAME);
                DBMS_OUTPUT.PUT_LINE ('START DATE IS: ' || X.STARTDATE);
                DBMS_OUTPUT.PUT_LINE ('END DATE IS: ' || X.ENDDATE);
            END LOOP;
        ELSE
            DBMS_OUTPUT.PUT_LINE ('PLEASE ENETER VALID COMPANY CODE');
        END IF;
    EXCEPTION
        WHEN NO_DATA_FOUND
        THEN
            DBMS_OUTPUT.PUT_LINE ('NO DATA FOUND. TRY AGAIN');
        WHEN OTHERS
        THEN
            DBMS_OUTPUT.PUT_LINE ('ERROR OCCURED IS: ' || SQLERRM);
    END p_generate_report;

    PROCEDURE p_get_rent (p_HNAME VARCHAR2)
    IS
        V_RENT        NUMBER;
        V_COST        NUMBER;
        V_STARTDATE   DATE;
        V_ENDDATE     DATE;
    BEGIN
        SELECT RENT
          INTO V_RENT
          FROM HALLS
         WHERE HNAME = p_HNAME;

        SELECT StartDate, EndDate
          INTO V_STARTDATE, V_ENDDATE
          FROM BOOKING
         WHERE HNAME = p_HNAME;

        V_COST := (V_ENDDATE - V_STARTDATE) * V_RENT;

        DBMS_OUTPUT.PUT_LINE ('THE COST OF HALL IS: ' || V_COST);
    END p_get_rent;
END Conference_Automation_test;
    
/



CREATE OR REPLACE TRIGGER update_trigger
    AFTER INSERT
    ON BOOKING
    FOR EACH ROW
BEGIN
    UPDATE HALLS
       SET NOOFBOOKINGS = NOOFBOOKINGS
     WHERE HNAME = :NEW.HNAME;
END;
/
    
