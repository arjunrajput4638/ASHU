uses gw.api.locale.DisplayKey
uses gw.util.debugsnapshot.DebugSnapshotUtil

CONDITION (aBContact : entity.ABContact):
  return (aBContact typeis ABPerson) and (aBContact.DateOfBirth != null
  and (gw.api.util.DateUtil.compareIgnoreTime(
      (aBContact as ABPerson).DateOfBirth,
      gw.api.util.DateUtil.currentDate()) > 0)

ACTION (aBContact : entity.ABContact, actions : gw.rules.Action):
  // Require that the DateOfBirth, if specified, is in the past.

  try {
    if (gw.api.util.DateUtil.compareIgnoreTime(
        (aBContact as ABPerson).DateOfBirth,
        gw.api.util.DateUtil.currentDate()) > 0) {
      
      // ✅ TRIGGER SNAPSHOT on validation failure
      DebugSnapshotUtil.captureSnapshot(aBContact)
      
      aBContact.reject(TC_LOADSAVE, "Date of Birth must be in the past", null, null)
    }
  } catch(e : Exception) {
    
    // ✅ TRIGGER SNAPSHOT on error
    DebugSnapshotUtil.captureOnError(aBContact, e)
    throw e
  }

END
