# AccuKnox_assignment
assignment 2 :
import subprocess
import logging
import os
from datetime import datetime
import smtplib
from email.mime.text import MIMEText
from email.mime.multipart import MIMEMultipart
# Configuration variables
SOURCE_DIR = "/path/to/source/directory" # Directory to be backed up
REMOTE_USER = "your-remote-username" # Username for remote server
REMOTE_HOST = "your-remote-host" # Hostname or IP of the remote server
REMOTE_DIR = "/path/to/remote/directory" # Destination directory on the remote server
LOG_FILE = "/path/to/logfile.log" # Path to log file
EMAIL_NOTIFICATION = True # Enable email notifications
EMAIL_RECIPIENT = "your-email@example.com" # Recipient email address
EMAIL_SENDER = "sender-email@example.com" # Sender email address
SMTP_SERVER = "smtp.example.com" # SMTP server address
SMTP_PORT = 587 # SMTP server port
SMTP_USER = "smtp-username" # SMTP server username
SMTP_PASSWORD = "smtp-password" # SMTP server password
# Set up logging
logging.basicConfig(filename=LOG_FILE, level=logging.INFO, format='%(asctime)s - %(levelname)s - %(message)s')
def log_message(message, level='info'): """ Log a message to the logfile and print to console. Parameters:
message (str): The message to log.
level (str): The log level ('info' or 'error'). """
if level == 'info':
logging.info(message)
elif level == 'error':
logging.error(message)
print(message)
def validate_paths(source, remote_dir): """ Validate the existence of the source directory. Parameters:
source (str): The source directory path. remote_dir (str): The remote directory path (unused in validation). Returns:
bool: True if the source directory exists, False otherwise
"""
if not os.path.isdir(source):
log_message(f"Source directory does not exist: {source}", level='error')
return False
return True
def send_email(subject, body): """ Send an email notification with the specified subject and body. Parameters:
subject (str): The email subject. body (str): The email body. """ msg = MIMEMultipart()
msg['From'] = EMAIL_SENDER
msg['To'] = EMAIL_RECIPIENT
msg['Subject'] = subject
msg.attach(MIMEText(body, 'plain'))
try:
server = smtplib.SMTP(SMTP_SERVER, SMTP_PORT)
server.starttls()
server.login(SMTP_USER, SMTP_PASSWORD)
text = msg.as_string()
server.sendmail(EMAIL_SENDER, EMAIL_RECIPIENT, text)
server.quit()
log_message("Email notification sent successfully.")
except Exception as e:
log_message(f"Failed to send email notification: {e}", level='error')
def perform_backup(source, remote_user, remote_host, remote_dir): """ Perform the backup using rsync. Parameters:
source (str): The source directory path. remote_user (str): The remote server username. remote_host (str): The remote server hostname or IP address. remote_dir (str): The remote directory path. Returns:
tuple: (bool, str) indicating success status and output message. """ command = [ "rsync", "-avz", "--delete", source, f"{remote_user}@{remote_host}:{remote_dir}"
]
try:
result = subprocess.run(command, check=True, capture_output=True, text=True)
log_message(f"Backup completed successfully:\n{result.stdout}")
return True, result.stdout
except subprocess.CalledProcessError as e:
log_message(f"Backup failed:\n{e.stderr}", level='error')
return False, e.stderr
def main(): """ Main function to perform backup and log the result. """ # Validate the source directory before attempting the backup
if not validate_paths(SOURCE_DIR, REMOTE_DIR):
log_message("Backup aborted due to validation error.", level='error')
if EMAIL_NOTIFICATION:
send_email("Backup Failed", "Backup aborted due to validation error.")
return
# Log the start of the backup process
log_message(f"Starting backup of {SOURCE_DIR} to
{REMOTE_USER}@{REMOTE_HOST}:{REMOTE_DIR}")
# Perform the backup and capture the success status and output
success, output = perform_backup(SOURCE_DIR, REMOTE_USER, REMOTE_HOST, REMOTE_DIR)
# Log the result and send an email notification if enabled
if success:
log_message("Backup completed successfully.")
if EMAIL_NOTIFICATION:
send_email("Backup Successful", f"Backup completed successfully.\n\n{output}")
else:
log_message("Backup failed.", level='error')
if EMAIL_NOTIFICATION:
send_email("Backup Failed", f"Backup failed.\n\n{output}")
if __name__ == "__main__":
main()
