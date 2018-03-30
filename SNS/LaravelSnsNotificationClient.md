```php

<?php

namespace App\Clients;

use Log;

class SnsNotificationClient
{
    protected $client;

    public function __construct()
    {
        $this->client = \App::make('aws')->createClient('sns');
    }

    /**
     * Register device
     * @param  string $os
     * @param  string $token
     * @param  string $arn
     * @return string      sns endpoint
     */
    public function register($os, $token, $arn)
    {
        // $os = strtolower($os);
        // switch ($os) {
        //     case 'ios':
        //         $arn = $arn;
        //         break;
        //     default:
        //         throw new Exception("NOT IMPLEMENTED", 1);
        // }

        // Register device
        $data = $this->client->createPlatformEndpoint([
            'PlatformApplicationArn' => $arn,
            'Token' => $token,
        ]);

        return $data['EndpointArn'];
    }

    /**
     * Sends a push notification
     * @param  string  $os
     * @param  string  $endpoint
     * @param  string  $message
     * @param  integer $badge
     * @param  array   $customData
     * @return void
     */
    public function send($os, $endpoint, $message, $badge = 1, $customData = [])
    {
        $payload = ['default' => $message];
        $os = strtolower($os);

        // Work around for emojis
        $message_placeholder = "SNS_MESSAGE_PLACEHOLDER";

        switch ($os) {
            case 'ios':
                $data = [
                    'aps' => [
                        'alert' => $message_placeholder,
                        'sound' => 'default',
                        'type' => $customData['type'],
                    ],
                ];
                break;
            default:
                return false;
        }

        $this->enable($endpoint);

        // Publish notification to device
        $this->client->publish([
            'TargetArn' => $endpoint,
            'Message' => $this->getPayload($data, $message, $badge, $customData),
            'MessageStructure' => 'json',
        ]);
    }

    /**
     * Format payload
     * @param  array $data
     * @param  string $message    text to display to user
     * @param  int $badge      number of unread notifications
     * @param  array $customData
     * @return string
     */
    private function getPayload($data, $message, $badge, $customData)
    {
        if (!is_null($badge)) {
            $data['aps']['badge'] = (int) $badge;
        }

        try {
            foreach ($customData as $key => $value) {
                $data['aps'][$key] = $value;
            }
        } catch (\Exception $e) {
            Log::warning($e->getMessage());
        }

        $apns = str_replace("SNS_MESSAGE_PLACEHOLDER", $message, json_encode($data));
        $apns = addslashes($apns);

        if (\App::environment() == 'production') {
            $payload = json_encode(["APNS" => "APS_KEY"]);
        } else {
            $payload = json_encode(["APNS_SANDBOX" => "APS_KEY"]);
        }

        return str_replace("APS_KEY", $apns, $payload);
    }

    /**
     * Enable endpoint
     * @param  string $endpoint SNS endpoint
     * @return void
     */
    public function enable($endpoint)
    {
        $this->client->setEndpointAttributes([
            'EndpointArn' => $endpoint,
            'Attributes' => [
                'Enabled' => 'true',
            ],
        ]);
    }

    /**
     * Delete endpoint
     * @param  string $endpoint SNS endpoint
     * @return void
     */
    public function deleteEndpoint($endpoint)
    {
        $this->client->deleteEndpoint([
            'EndpointArn' => $endpoint,
        ]);
    }
}

```
