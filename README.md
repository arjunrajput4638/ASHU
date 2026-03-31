package gw.util.debugsnapshot

uses org.slf4j.LoggerFactory
uses java.io.FileWriter
uses java.io.File
uses java.util.Date

class DebugSnapshotUtil {

  static var _logger = LoggerFactory.getLogger("DebugSnapshot")
  
  // ✅ Change this path to your TrainingApp folder
  static final var SNAPSHOT_DIR = "C:/Users/arthakur/TrainingAppV10/TrainingAppV10/GW10/TrainingApp/snapshots"

  static function captureSnapshot(contact : ABContact) : String {
    var sb = new StringBuilder()
    
    // --- Build snapshot data ---
    sb.append("=== DEBUG SNAPSHOT ===\n")
    sb.append("Timestamp  : " + new Date() + "\n")
    sb.append("Entity     : ABContact\n")
    sb.append("PublicID   : " + (contact.PublicID?:"NOT YET SAVED") + "\n")
    sb.append("Name       : " + contact.DisplayName + "\n")
    sb.append("Subtype    : " + contact.Subtype + "\n")

    // --- Extra fields ---
    if (contact typeis ABPerson) {
      var p = contact as ABPerson
      sb.append("DOB        : " + (p.DateOfBirth?:"N/A") + "\n")
      sb.append("CellPhone  : " + (p.CellPhone?:"N/A") + "\n")
      sb.append("HomePhone  : " + (contact.HomePhone?:"N/A") + "\n")
      sb.append("WorkPhone  : " + (contact.WorkPhone?:"N/A") + "\n")
    }

    // --- Address ---
    if (contact.PrimaryAddress != null) {
      var addr = contact.PrimaryAddress
      sb.append("Address    : " + (addr.AddressLine1?:"") 
                + ", " + (addr.City?:"")
                + ", " + (addr.State?:"")
                + ", " + (addr.PostalCode?:"") + "\n")
    } else {
      sb.append("Address    : N/A\n")
    }

    sb.append("======================\n")
    var snapshot = sb.toString()

    // --- Log it ---
    _logger.info(snapshot)

    // --- Save to files ---
    saveToFile(contact, snapshot)

    return snapshot
  }

  static function captureOnError(contact : ABContact, e : Exception) : String {
    var sb = new StringBuilder()
    sb.append("=== ERROR SNAPSHOT ===\n")
    sb.append("Timestamp  : " + new Date() + "\n")
    sb.append("Entity     : ABContact\n")
    sb.append("PublicID   : " + (contact.PublicID?:"NOT YET SAVED") + "\n")
    sb.append("Name       : " + contact.DisplayName + "\n")
    sb.append("Subtype    : " + contact.Subtype + "\n")
    sb.append("Error      : " + e.Message + "\n")
    sb.append("StackTrace : " + e.StackTraceAsString + "\n")
    sb.append("======================\n")
    var snapshot = sb.toString()
    _logger.error(snapshot)
    saveToFile(contact, snapshot)
    return snapshot
  }

  private static function saveToFile(contact : ABContact, snapshot : String) {
    try {
      // --- Create snapshots directory ---
      var dir = new File(SNAPSHOT_DIR)
      if (!dir.exists()) {
        dir.mkdirs()
      }

      var timestamp = new java.text.SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date())
      var name = (contact.DisplayName?:"unknown").replaceAll(" ", "_")
      var baseName = SNAPSHOT_DIR + "/" + name + "_" + timestamp

      // --- Save JSON ---
      var json = buildJSON(contact, snapshot)
      writeFile(baseName + ".json", json)

      // --- Save HTML ---
      var html = buildHTML(contact, snapshot)
      writeFile(baseName + ".html", html)

      // --- Save Markdown ---
      var md = buildMarkdown(contact, snapshot)
      writeFile(baseName + ".md", md)

      _logger.info("Snapshots saved to: " + baseName)

    } catch (e : Exception) {
      _logger.error("Failed to save snapshot file: " + e.Message)
    }
  }

  private static function writeFile(path : String, content : String) {
    var fw = new FileWriter(path)
    fw.write(content)
    fw.close()
  }

  private static function buildJSON(contact : ABContact, raw : String) : String {
    var sb = new StringBuilder()
    sb.append("{\n")
    sb.append("  \"timestamp\": \"" + new Date() + "\",\n")
    sb.append("  \"entity\": \"ABContact\",\n")
    sb.append("  \"publicId\": \"" + (contact.PublicID?:"null") + "\",\n")
    sb.append("  \"name\": \"" + (contact.DisplayName?:"") + "\",\n")
    sb.append("  \"subtype\": \"" + contact.Subtype + "\",\n")
    if (contact typeis ABPerson) {
      var p = contact as ABPerson
      sb.append("  \"dob\": \"" + (p.DateOfBirth?:"null") + "\",\n")
      sb.append("  \"cellPhone\": \"" + (p.CellPhone?:"null") + "\",\n")
      sb.append("  \"homePhone\": \"" + (contact.HomePhone?:"null") + "\",\n")
    }
    if (contact.PrimaryAddress != null) {
      sb.append("  \"address\": \"" + (contact.PrimaryAddress.AddressLine1?:"") + "\"\n")
    } else {
      sb.append("  \"address\": \"null\"\n")
    }
    sb.append("}")
    return sb.toString()
  }

  private static function buildHTML(contact : ABContact, raw : String) : String {
    return "<html><body style='font-family:Arial'>"
      + "<h2>Debug Snapshot</h2>"
      + "<table border='1' cellpadding='5'>"
      + "<tr><td><b>Timestamp</b></td><td>" + new Date() + "</td></tr>"
      + "<tr><td><b>Entity</b></td><td>ABContact</td></tr>"
      + "<tr><td><b>PublicID</b></td><td>" + (contact.PublicID?:"NOT YET SAVED") + "</td></tr>"
      + "<tr><td><b>Name</b></td><td>" + (contact.DisplayName?:"") + "</td></tr>"
      + "<tr><td><b>Subtype</b></td><td>" + contact.Subtype + "</td></tr>"
      + (contact typeis ABPerson 
          ? "<tr><td><b>DOB</b></td><td>" + ((contact as ABPerson).DateOfBirth?:"N/A") + "</td></tr>"
          : "")
      + "</table></body></html>"
  }

  private static function buildMarkdown(contact : ABContact, raw : String) : String {
    var sb = new StringBuilder()
    sb.append("# Debug Snapshot\n\n")
    sb.append("| Field | Value |\n")
    sb.append("|-------|-------|\n")
    sb.append("| Timestamp | " + new Date() + " |\n")
    sb.append("| Entity | ABContact |\n")
    sb.append("| PublicID | " + (contact.PublicID?:"NOT YET SAVED") + " |\n")
    sb.append("| Name | " + (contact.DisplayName?:"") + " |\n")
    sb.append("| Subtype | " + contact.Subtype + " |\n")
    if (contact typeis ABPerson) {
      var p = contact as ABPerson
      sb.append("| DOB | " + (p.DateOfBirth?:"N/A") + " |\n")
      sb.append("| CellPhone | " + (p.CellPhone?:"N/A") + " |\n")
    }
    return sb.toString()
  }

}
