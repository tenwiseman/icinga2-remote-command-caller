# icinga2-remote-command-caller
When using Icinga2 in a distributed master configuration, this icinga function is a useful wrapper for executing both PowerShell and CMD commands in the remote environment, passing run-time parameters and optionally running a postexit script on the command's returned results

function parameters:

 	lang(string) : Shell environment language. either 'powershell' or 'cmd'.
 		Add the string 'commandline' to view the actual command line that would be sent
 
 	executable(string) : full path name of the external command.
 		For CMD, this should be wrapped in double quotes \"
 		For PowerShell, this should be wrapped in single '
 
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
 
  These are some examples of postexit scripts, they are executed remote side
  so are written in the same environment language.
 
  Powershell:
  
 	# change 'unknown' status to 'warning' status
  	;if ($lastexitcode -eq 3) {exit 1} else {exit $lastexitcode}"
 
  CMD:
 
 	# change 'unknown' status to 'warning' status
 	& IF ERRORLEVEL 3 EXIT /B 1 ELSE EXIT /B !ERRORLEVEL 
 
