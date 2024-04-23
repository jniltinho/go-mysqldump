# Go MYSQL Dump
Create MYSQL dumps in Go without the `mysqldump` CLI as a dependancy.

### Simple Example
```go
package main

import (
	"database/sql"
	"flag"
	"fmt"
	"os"

	"github.com/go-sql-driver/mysql"
	"github.com/jamf/go-mysqldump"
)

type Options struct {
	User      string
	Passwd    string
	Host      string
	DBName    string
	Net       string
	ParseTime bool
	Charset   string
}

func main() {

	opt := ParseOptions()

	// Open connection to database
	config := mysql.NewConfig()
	config.User = opt.User
	config.Passwd = opt.Passwd
	config.DBName = opt.DBName
	config.Net = opt.Net
	config.Addr = opt.Host
	config.ParseTime = opt.ParseTime
	config.Collation = opt.Charset

	dumpDir := "dumps"
	dumpFilenameFormat := fmt.Sprintf("%s-20060102_1504", config.DBName) // accepts time layout string and add .sql at the end of file

	if err := os.MkdirAll(dumpDir, 0755); err != nil {
		fmt.Println("Error mkdir:", err)
		return
	}

	db, err := sql.Open("mysql", config.FormatDSN())
	if err != nil {
		fmt.Println("Error opening database:", err)
		return
	}

	// Register database with mysqldump
	dumper, err := mysqldump.Register(db, dumpDir, dumpFilenameFormat)
	if err != nil {
		fmt.Println("Error registering databse:", err)
		return
	}

	// Dump database to file
	if err := dumper.Dump(); err != nil {
		fmt.Println("Error dumping:", err)
		return
	}
	if file, ok := dumper.Out.(*os.File); ok {
		fmt.Println("File is saved to", file.Name())
	} else {
		fmt.Println("It's not part of *os.File, but dump is done")
	}

	// Close dumper, connected database and file stream.
	dumper.Close()
}

func ParseOptions() *Options {
	opt := &Options{}
	flag.StringVar(&opt.Host, "host", "127.0.0.1:3306", "Host")
	flag.StringVar(&opt.Net, "net", "tcp", "Net")
	flag.BoolVar(&opt.ParseTime, "parseTime", true, "ParseTime")
	flag.StringVar(&opt.Charset, "charset", "utf8mb4_unicode_ci", "Charset")
	flag.StringVar(&opt.User, "user", "root", "User")
	flag.StringVar(&opt.Passwd, "pass", "", "Password")
	flag.StringVar(&opt.DBName, "db", "test", "Database")

	flag.Usage = func() {
		fmt.Fprintln(os.Stderr, "------------------------------------------------------------")
		fmt.Fprintln(os.Stderr, "Usage:")
		flag.PrintDefaults()
		fmt.Fprintf(os.Stderr, "\nExample: %s -host=127.0.0.1:3306 -user=root -pass=root -db=test\n", os.Args[0])
		fmt.Fprintln(os.Stderr, "------------------------------------------------------------")
	}

	if len(os.Args) == 1 {
		flag.Usage()
		os.Exit(1)
	}

	flag.Parse()
	return opt
}

```

[![GoDoc](https://godoc.org/github.com/jamf/go-mysqldump?status.svg)](https://godoc.org/github.com/jamf/go-mysqldump)
[![Build Status](https://travis-ci.org/jamf/go-mysqldump.svg?branch=master)](https://travis-ci.org/jamf/go-mysqldump)
