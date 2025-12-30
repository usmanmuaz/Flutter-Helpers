# Usman Muaz
# SENIOR FLUTTER DEVELOPER

============================================================================================================
# Step 1 - FCM Working Payload for Backend to Send Notification
============================================================================================================

 Future<void> sendNotificationWithImage({
    String? fcmToken,
    required String title,
    required String body,
    String? imageUrl,
  }) async {
  
    try {
    
      imageUrl = "https://picsum.photos/400/300";
      String accessToken = await _getAccessToken();
      print("🔑 Access Token obtained");
      
      final headers = {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer $accessToken',
      };
     
      final payload = {
        "message": {
          "token": fcmToken,
          "data": {
            "title": title,
            "body": body,
            if (imageUrl != null && imageUrl.isNotEmpty) "image": imageUrl,
            "type": "notification",
          },
          "android": {
            "priority": "high",
            "notification": {
              "title": title,
              "body": body,
              if (imageUrl != null && imageUrl.isNotEmpty) "image": imageUrl,
              "channel_id": "high_importance_channel",
            },
          },
          "apns": {
            // APNS Headers
            "headers": {
              "apns-push-type":
                  "alert", // ← ADDED: Required for alert notifications
              "apns-priority":
                  "10", // ← FIXED: "10" for immediate delivery (not "5")
            },
            "payload": {
              "aps": {
                "alert": {
                  "title": title,
                  "body": body,
                },
                "mutable-content":
                    1, // ← FIXED: Must be 1 (not 0) to trigger NSE
                "sound": "default",
              }
            },
            // FCM Options - OUTSIDE payload (for image)
            if (imageUrl != null && imageUrl.isNotEmpty)
              "fcm_options": {
                "image": imageUrl,
              }
          },
        },
      };

      print("📤 Sending notification payload:");
      print(jsonEncode(payload));

      final response = await http.post(
        Uri.parse(
            'https://fcm.googleapis.com/v1/projects/projectFirebaseId/messages:send'),
        headers: headers,
        body: jsonEncode(payload),
      );

      if (response.statusCode == 200) {
        print('✅ Notification sent successfully');
        print('Response: ${response.body}');
      } else {
        print('❌ Failed to send notification');
        print('Status Code: ${response.statusCode}');
        print('Response: ${response.body}');
      }
    } catch (e) {
      print('❌ Error sending notification: $e');
    }
  }

============================================================================================================
============================================================================================================
# Step 2 - CREATE NSE with Objective C Language
============================================================================================================

* https://firebase.flutter.dev/docs/messaging/apple-integration/

============================================================================================================
============================================================================================================
# Step 3 - In Pod File 
============================================================================================================
target 'ImageNotification' do
  use_frameworks!
  use_modular_headers!
  
  pod 'Firebase/Messaging'
  pod 'FirebaseMessaging'
end

============================================================================================================
============================================================================================================
# Step 4 - In XCode
============================================================================================================

* Open XCode > Target Runner > Build Phases > Embed Foundation Extension > Uncheck Copy only when Installing
  
