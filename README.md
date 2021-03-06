# icinga2-remote-command-caller

by Tenwiseman, June 2020

When using Icinga2 in a distributed master configuration, this icinga function is a useful wrapper for executing both PowerShell and CMD commands in the remote environment, passing run-time parameters and optionally running a postexit script to modify 
any returned results. 

remote_command_caller() parameters:

 	lang(string) : Shell environment language. Either 'powershell' or 'cmd'.
 		Add the string 'commandline' to debug view the actual command line that would be sent.
 
 	executable(string) : full path name of the external command.
 		For CMD, this should be wrapped in double quotes \"
 		For PowerShell, this should be wrapped in single quotes '
 
 	postexit(string) : the postexit script string, see below
  
 
  The following is an CheckCommand usage example 
 
 	object CheckCommand "cmd-perfmon" {
 
        	vars.arguments = {
                	"-w" = "$perfmon_win_warn$"
                	"-c" = "$perfmon_win_crit$"
                	"-P" = "\"$perfmon_win_counter$\""
                	"--performance-wait" = "$perfmon_win_wait$"
                	"--fmt-countertype" = "$perfmon_win_type$"
                	"--perf-syntax" = "$perfmon_win_syntax$"
        	}
 
        	vars.perfmon_win_wait = 1000
        	vars.perfmon_win_type = "double"
 
        	command = remote_command_caller(
                	"cmd",
                	"\"C:\\Program Files\\ICINGA2\\sbin\\check_perfmon.exe\"",
                	" & IF ERRORLEVEL 3 EXIT /B 1 ELSE EXIT /B !ERRORLEVEL"
        	)
 
 	}
 
  These are some examples of optional postexit scripts.
  
  As these are sequentially executed remote side, they should be written in the same
  environment language, and preceded as shown with a either a semi-colon or ampersand character.
 
  Powershell:
  
 	# change 'unknown' status to 'warning' status.
    ;if ($lastexitcode -eq 3) {exit 1} else {exit $lastexitcode}"
 
  CMD:
 
 	# change 'unknown' status to 'warning' status
 	& IF ERRORLEVEL 3 EXIT /B 1 ELSE EXIT /B !ERRORLEVEL"
 
