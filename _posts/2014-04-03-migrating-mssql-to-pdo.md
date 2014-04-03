---
layout: post
title: Migrating Php 5.2 apps using mssql to PDO functions 
---

## Preface
As mssql php extension is deprecated in Php 5.3 and above, the only way to keep using its functions was sticking with a 5.2 Php, but it's very outdated now and it's not supported any more. I'm just sick of Php 5.2 and really want to upgrade to latest stable version, 5.5. Luckly found a wrapper script to override those funtions that are deprecated in latest Php versions and call PDO functions for database access without modifying most of app's code.

## Problems and beneficts
The old set of funtions (the ones included in mssql php extension) where kind of broken, for example if you execute two stacked select statements, the first one works like a charm, but you can't access the second results dataset, actually if you run another query, anything, it will fail. Yes I know, that is ugly.

PDO has a lot of beneficts, the gratest thing is how exception are handled (now raiserror statement returns the error to the app!) but as I was testing the app with Php 5.5 I noticed that a lot of PDO exceptions where raised, and that was fine, because before the deprecated extension was silently failing or hiding warnings. So this great improvement for the app, meant a lot of time fixing those exceptions.

The new php version performance is quite good, I haven't measure it properly, but general server usage metrics are lower than before and the amount of hits and userbase are the same that before.

## Code
Yes, finally the most interesting part. This code is licenced under GPL v3. Use it, and hack it, it would be nice to see more changes or improvements. Any comment would be great too.

{% highlight php %}
<?php

if (! extension_loaded('mssql')) {
    //Return an associative array. Used on mssql_fetch_array()'s result_type parameter.
    define('MSSQL_ASSOC', '1');

    //Return an array with numeric keys. Used on mssql_fetch_array()'s result_type parameter.
    define('MSSQL_NUM', '2');

    //Return an array with both numeric keys and keys with their field name. This is the default value for mssql_fetch_array()'s result_type parameter.
    define('MSSQL_BOTH', '3');

    //Indicates the 'TEXT' type in MSSQL, used by mssql_bind()'s type parameter.
    define('SQLTEXT', '35');

    //Indicates the 'VARCHAR' type in MSSQL, used by mssql_bind()'s type parameter.
    define('SQLVARCHAR', '39');

    //Indicates the 'CHAR' type in MSSQL, used by mssql_bind()'s type parameter.
    define('SQLCHAR', '47');

    //Represents one byte, with a range of -128 to 127.
    define('SQLINT1', '48');

    //Represents two bytes, with a range of -32768 to 32767.
    define('SQLINT2', '52');

    //Represents four bytes, with a range of -2147483648 to 2147483647.
    define('SQLINT4', '56');

    //Indicates the 'BIT' type in MSSQL, used by mssql_bind()'s type parameter.
    define('SQLBIT', '50');

    //Represents an four byte float.
    define('SQLFLT4', '59');

    //Represents an eight byte float.
    define('SQLFLT8', '62');

    class MSSQL_PDO extends PDO {
        public function __construct($dsn, $username="", $password="", $driver_options=array()) {
            parent::__construct($dsn,$username,$password, $driver_options);
            if (empty($driver_options[PDO::ATTR_STATEMENT_CLASS])) {
                $this->setAttribute(PDO::ATTR_STATEMENT_CLASS, array('MSSQL_PDOStatement', array($this)));
            }
        }
    }
    class MSSQL_PDOStatement extends PDOStatement {
        //public $_all = null;
        public $dbh;
        protected function __construct($dbh) {
            $this->dbh = $dbh;
        }

    }

    function mssql_last_link($link_identifier = null) {
        static $last = null;
        if ($link_identifier) {
            $last = $link_identifier;
        }
        return $last;
    }

    function mssql_bind($stmt, $param_name, &$var, $type, $is_output = false, $is_null = false, $maxlen = -1) {
        $mssql_type_map = array(
                SQLTEXT => PDO::PARAM_LOB,
                SQLVARCHAR => PDO::PARAM_STR,
                SQLCHAR => PDO::PARAM_STR,
                SQLINT1 => PDO::PARAM_INT,
                SQLINT2 => PDO::PARAM_INT,
                SQLINT4 => PDO::PARAM_INT,
                SQLBIT =>  PDO::PARAM_BOOL,
                SQLFLT4 => PDO::PARAM_INT,
                SQLFLT8 => PDO::PARAM_INT,
        );
        $var = $is_null?null:$var;
        $ret = $stmt->bindParam($param_name, $var, $mssql_type_map[$type], $is_output?$maxlen:($maxlen<0?null:$maxlen));
        return $ret;
    }

/*
	@todo: write close connection function
    function mssql_close() {

    }
*/

    function mssql_connect($servername, $username, $password, $new_link = false) {
        $pdo = new PDO('sqlsrv:Server='.$servername .';', $username, $password);
        
        $pdo->setAttribute(PDO::SQLSRV_ENCODING_SYSTEM, 0);

        // cursor scroll is set to allow driver to count rows (mssql_num_rows)
        $pdo->setAttribute(PDO::ATTR_CURSOR, PDO::CURSOR_SCROLL);
        $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        $pdo->setAttribute(PDO::ATTR_STATEMENT_CLASS, array('MSSQL_PDOStatement', array($pdo)));

        mssql_last_link($pdo);

        return $pdo;
    }

    function mssql_pconnect($servername, $username, $password, $new_link = false) {
        $pdo = mssql_connect($servername, $username, $password, $new_link);

        // @todo this may not work except at connection time.
        // this wont work in fast-cgi or pdo_sqlsrv in anycase.
        // left here as a woulda-shoulda-cant
        //$pdo->setAttribute(PDO::ATTR_PERSISTENT, true);
        return $pdo;
    }

    function mssql_execute($stmt, $skip_results = false) {
        return $stmt->execute();
    }


    function mssql_fetch_array($result, $result_type = MSSQL_BOTH) {
        $mssql_result_type = array(
                MSSQL_ASSOC => PDO::FETCH_ASSOC,
                MSSQL_NUM => PDO::FETCH_NUM,
                MSSQL_BOTH => PDO::FETCH_BOTH
        );
        return $result->fetch($mssql_result_type[$result_type]);
    }

    function mssql_fetch_assoc($result) {
        return $result->fetch(PDO::FETCH_ASSOC);
    }

    function mssql_fetch_object($result) {
        return $result->fetch(PDO::FETCH_OBJ);
    }

    function mssql_fetch_row($result) {
        return $result->fetch();
    }

    function mssql_free_result($result) {
        return $stmt->closeCursor();
    }

    function mssql_free_statement($stmt) {
        return $stmt->closeCursor();
    }

    function mssql_get_last_message() {
        $link_identifier = mssql_last_link();

        $errors = $link_identifier->errorInfo();
        return $errors[2];
    }

    function mssql_init($sp_name, $link_identifier = null) {
        if (is_null($link_identifier)) {
            $link_identifier = mssql_last_link();
        }

        return $link_identifier->prepare('exec '.$sp_name);
    }

    function mssql_next_result($result_id) {
        return $result_id->nextRowset();
    }

    function mssql_num_fields($result) {
        return $result->columnCount();
    }

    function mssql_num_rows($result) {
        return $result->rowCount();
    }

    function mssql_query($query, $link_identifier = null, $batch_size = 0) {
        if (is_null($link_identifier)) {
            $link_identifier = mssql_last_link();
        }

		//$driver_options = array(PDO::SQLSRV_ATTR_ENCODING => PDO::SQLSRV_ENCODING_UTF8)
        $driver_options = array(PDO::ATTR_CURSOR => PDO::CURSOR_SCROLL)
        $stmt = $link_identifier->prepare($query, $driver_options);
        $return = $stmt->execute();

        // Match mssql_query behavior
        if ($return) {
            $rows = $stmt->rowCount();
            $link_identifier->lastAffected = $rows;

            if (is_null($rows)) {
                $return = true;
            } else {
                $return = $stmt;
            }
        }
        return $return;

    }

    function mssql_data_seek($result_identifier, $row_number) {
        $row = $result_identifier->fetch(PDO::FETCH_BOTH, PDO::FETCH_ORI_ABS, $row_number);

        // Rewind - CI always does a data_seek(0) as a rewind, but that offsets the cursor,
        // we have to move to -1 so the next fetch will return the first piece of data
        // @todo rather than resetting, it would be better to restore the old position
        // if we can't fetch that data somehow then track the position in the statement class
        $row = $result_identifier->fetch(PDO::FETCH_BOTH, PDO::FETCH_ORI_ABS, -1);

        return $row;
    }

    function mssql_result($resource, $row, $field) {
        $data_row = mssql_data_seek($resource, $row);
        return $data_row[$field];
    }


    function mssql_rows_affected($link_identifier = null) {
        if (is_null($link_identifier)) {
            $link_identifier = mssql_last_link();
        }
        return  $link_identifier->lastAffected;

    }

    // @todo try/catch return false on failure
    function mssql_select_db($database_name, $link_identifier = null) {
        if (is_null($link_identifier)) {
            $link_identifier = mssql_last_link();
        }
        $affected = $link_identifier->exec('USE '.$database_name);
        $link_identifier->lastAffected = $affected;

        return true;
    }
}
?>
{% endhighlight %}