How to Create and Grant Permissions on Directories


Oracle Apex provides a way to create directories and grant permissions to them. This can be useful for storing files that need to be accessible to specific users or groups. To create a directory, you will need to specify a name and path for the directory. You can then grant permissions to users or groups on the directory. The permissions that you can grant include "READ", "WRITE", and "EXECUTE".

Code - 

CREATE OR REPLACE DIRECTORY DIRECTORY_NAME AS 'C:\FOLDER_LOCATION';
GRANT READ, WRITE ON DIRECTORY DIRECTORY_NAME TO USERNAME;






 👉Create a Stored Procedure for Converting BLOBs to Files in a Database

A BLOB (Binary Large Object) is a data type in Oracle that can store large binary data, such as images, audio, and video. A file is a named collection of data that is stored on a disk or other storage device.

The blob_to_file procedure converts a BLOB to a file. You can use this procedure to insert an image in a directory from a database.

Code - 
 
CREATE OR REPLACE PROCEDURE blob_to_file (
   p_blob       IN OUT NOCOPY BLOB,
   p_dir        IN            VARCHAR2,
   p_filename   IN            VARCHAR2)
AS
   l_file       UTL_FILE.FILE_TYPE;
   l_buffer     RAW (32767);
   l_amount     BINARY_INTEGER := 32767;
   l_pos        INTEGER := 1;
   l_blob_len   INTEGER;
BEGIN
   l_blob_len := DBMS_LOB.getlength (p_blob);
   -- Open the destination file.
   l_file :=
      UTL_FILE.fopen (p_dir,
                      p_filename,
                      'WB',
                      32767);

   -- Read chunks of the BLOB and write them to the file until complete.
   WHILE l_pos <= l_blob_len
   LOOP
      DBMS_LOB.read (p_blob,
                     l_amount,
                     l_pos,
                     l_buffer);
      UTL_FILE.put_raw (l_file, l_buffer, TRUE);
      l_pos := l_pos + l_amount;
   END LOOP;

   -- Close the file.
   UTL_FILE.fclose (l_file);
EXCEPTION
   WHEN OTHERS
   THEN
      -- Close the file if something goes wrong.
      IF UTL_FILE.is_open (l_file)
      THEN
         UTL_FILE.fclose (l_file);
      END IF;

      RAISE;
/* WHEN UTL_FILE.invalid_operation THEN  dbms_output.PUT_LINE('cannot open file invalid name');
WHEN UTL_FILE.read_error THEN dbms_output.PUT_LINE('cannot be reads);
WHEN no_data_found THEN dbms_output.PUT_LINE('end of file');
UTL_FILE.fclose(l_file);
RAISE;*/
END blob_to_file;




 👉Insert an Image in a Directory from an APEX Application

To insert an image in a directory from a database using the blob_to_file procedure, you can use the following steps:

Code - 
 
DECLARE
   l_blob   BLOB;
BEGIN
   SELECT BLOB_CONTENT
     INTO l_blob
     FROM apex_application_temp_files
    WHERE NAME = :P1_IMAGE;

   blob_to_file (p_blob       => l_blob,
                 p_dir        => 'DIRECTORY_NAME',
                 p_filename   => PARAMETER || '.png');
END;






  👉The query for an Image from a Directory using Oracle APEX

Code - 

SELECT prod_id,
       prod_name,
       unit,
       rate,
          '<img src="http://localhost:8080/i/DIRECTORY_NAME/'
       || Parameter
       || '.png" width=100px height=100px </img>'
          image
  FROM PRODUCT_INFO

