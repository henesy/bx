# bx - Parse Burp XML

A tool to parse Burp suite HTTP proxy history XML files. 

Written in [Go](https://golang.org).

## Dependencies

- https://github.com/henesy/burpxml

## Build

	go build

## Usage

	; bx -h
	Usage of bx:
	-R    omit responses in CSV (as they may corrupt output in excel)
	-c    emit XML as CSV only
	-d    decode base64 bodies (may corrupt output)
	-g    emit XML as valid Go syntax only
	-i string
			input file name (rather than first argument)
	-j    emit XML as JSON only
	-o string
			output file name (rather than stdout)
	-r    omit requests in CSV (as they may corrupt output in excel)
	-s    read from stdin (rather than first argument)
	;

The file name `-` may be used to specify usage of stdin/stdout. 

## Examples

Convert XML output to a file as JSON with requests/responses decoded:

	; bx -d -j -o history.json history.xml
	; 

Get all hosts queried, omitting requests/responses from the output:

	; bx -r -R -c -i history.xml | awk -F ',' '{print $3}' | sort | uniq
	login.live.com
	outlook.office365.com
	;

Create a JSON subset of the XML data consisting of an array of paths:

	; bx -j -i history.xml | jq '.Items[] | {path: .Path}'
	{
	"path": "/async/bar"
	}
	{
	"path": "/ListStuff"
	}
	{
	"path": "/async/foo"
	}
	;

Create a JSON version of the XML data, exposing the history as a file system rooted at `$HOME/n/json` using [jsonfs](https://github.com/droyo/jsonfs) and [9pfs](https://github.com/mischief/9pfs):

	; bx -i history.xml -o history.json -j -d
	; jsonfs history.json &
	; 9pfs -p 5640 127.0.0.1 $HOME/n/json &
	; cd $HOME/n/json
	; ls
	Items
	; ls Items/1
	Comment
	Extension
	Host/
	MimeType
	Path
	Port
	Protocol
	Request/
	Response/
	ResponseLength
	Status
	Time
	Url
	; grep -i office Items/*/Host/Name
	2/Host/Name:outlook.office365.com
	7/Host/Name:outlook.office365.com
	; 

Note that the above strategy is particularly useful for accessing requests/responses as needed and permits trivial shell scripting over the file system. 

