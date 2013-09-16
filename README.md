Session-Handler
===============
<?php
class SessionDB {

    private $data = null;
    private $session_id = null;
    private $minutes_to_expire = 3600; // TIME TO MAINTAIN DATA ON DB

    public function __construct(){
      global $SESSION;

      $this->cookie = ( object ) $_COOKIE;

      $tbl = 'tb_session_db';

      if ( isset( $this->cookie->session_id ){

          $this->session_id = $this->cookie->session_id;

      } else {

            $this->session_id = md5(microtime().rand(1,9999999999999999999999999)); // GENERATE A RANDOM ID
            setcookie('session_id',$this->session_id);

            $sql = $this->db->insert( $tbl, $this->ci->array('session_id', 'updated_on'), $this->ci->array($this->session_id, NOW() ) );
      }

      $sql = $this->db->query("SELECT `value` FROM `tb_session_db` WHERE `session_id`='{$this->session_id}'");

      $this->data = unserialize(mysql_result($sql, 0, 'value'));
      $SESSION = $this->data;
    }

    private function expire(){
          $date_to_delete = date("Y-m-d H:i:s", time()-60*$this->minutes_to_expire);
          $sql = $this->db->delete( $tbl, $this->ci->array('session_id' => $this->session_id  ) );
          if( $sql ){
              return $sql;
          }
    }

    public function __destruct(){
      global $SESSION;
          $this->data = serialize($SESSION);
          $sql = $this->db->update( $tbl, $this->ci->array('value'=>$this->data, 'updated_on'=> NOW() ), $this->ci->array('session_id'=>$this->session_id) );
          $this->expire();
    }
  }
?>
