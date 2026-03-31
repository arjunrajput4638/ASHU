package gw.util.debugsnapshot

uses org.slf4j.LoggerFactory

class DebugSnapshotUtil {

  static var _logger = LoggerFactory.getLogger("DebugSnapshot")

  static function captureSnapshot(contact : ABContact) : String {
    var sb = new StringBuilder()
    sb.append("=== DEBUG SNAPSHOT ===\n")
    sb.append("Timestamp  : " + new java.util.Date() + "\n")
    sb.append("Entity     : ABContact\n")
    sb.append("PublicID   : " + contact.PublicID + "\n")
    sb.append("Name       : " + contact.DisplayName + "\n")
    sb.append("Subtype    : " + contact.Subtype + "\n")
    sb.append("======================\n")
    var snapshot = sb.toString()
    _logger.info(snapshot)
    return snapshot
  }

  static function captureOnError(contact : ABContact, e : Exception) : String {
    var sb = new StringBuilder()
    sb.append("=== ERROR SNAPSHOT ===\n")
    sb.append("Timestamp  : " + new java.util.Date() + "\n")
    sb.append("Entity     : ABContact\n")
    sb.append("PublicID   : " + contact.PublicID + "\n")
    sb.append("Name       : " + contact.DisplayName + "\n")
    sb.append("Error      : " + e.Message + "\n")
    sb.append("======================\n")
    var snapshot = sb.toString()
    _logger.error(snapshot)
    return snapshot
  }

}
