ZaloAPI
class ZaloAPI {
    constructor() {
        this.accessToken = null; // Global access token
        this.cache = {}; // Cache for frequently accessed data
    }
    // Extension metadata and blocks
    getInfo() {
        return {
            id: 'zaloAPI',
            name: 'ZaloAPI',
            blocks: [
                // Authentication
                {
                    opcode: 'login',
                    blockType: Scratch.BlockType.COMMAND,
                    text: 'Log in with auth code [AUTH_CODE]',
                    arguments: {
                        AUTH_CODE: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: 'Enter auth code'
                        }
                    }
                },
                {
                    opcode: 'getUserInfo',
                    blockType: Scratch.BlockType.REPORTER,
                    text: 'Get user information'
            },
                 // Messaging
                {
                    opcode: 'sendMessageToUser',
                    blockType: Scratch.BlockType.COMMAND,
                    text: 'Send message [MESSAGE] to user [USER_ID]',
                    arguments: {
                        MESSAGE: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: 'Hello!'
                        },
                        USER_ID: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: '123456789'
                        }
                    }
                },
                {
                    opcode: 'sendImageToUser',
                    blockType: Scratch.BlockType.COMMAND,
                    text: 'Send image [IMAGE_URL] to user [USER_ID]',
                    arguments: {
                        IMAGE_URL: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: 'https://example.com/image.jpg'
                        },
                        USER_ID: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: '123456789'
                        }
                    }
                },
               // Group Messaging
                {
                    opcode: 'sendMessageToGroup',
                    blockType: Scratch.BlockType.COMMAND,
                    text: 'Send message [MESSAGE] to group [GROUP_ID]',
                    arguments: {
                        MESSAGE: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: 'Hello group!'
                        },
                        GROUP_ID: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: '987654321'
                        }
                    }
                },
             // Friend Management
                {
                    opcode: 'getFriendList',
                    blockType: Scratch.BlockType.REPORTER,
                    text: 'Get friend list'
                },
                 // Media Handling
                {
                    opcode: 'sendVideoToUser',
                    blockType: Scratch.BlockType.COMMAND,
                    text: 'Send video [VIDEO_URL] to user [USER_ID]',
                    arguments: {
                        VIDEO_URL: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: 'https://example.com/video.mp4'
                        },
                        USER_ID: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: '123456789'
                        }
                    }
                },
                   // Custom API
                {
                    opcode: 'customApiRequest',
                    blockType: Scratch.BlockType.REPORTER,
                    text: 'Send custom request to [API_ENDPOINT]',
                    arguments: {
                        API_ENDPOINT: {
                            type: Scratch.ArgumentType.STRING,
                            defaultValue: 'https://openapi.zalo.me/v2.0/some_endpoint'
                        }
                    }
                }
            ]
        };
    }
             // Unified API Call Function
    async apiCall(endpoint, method = 'GET', body = null) {
        if (!this.accessToken) {
            console.error('User is not logged in');
            return 'Not logged in';
        }
        const headers = { 'access_token': this.accessToken };
        if (body) headers['Content-Type'] = 'application/json';
     try {
            const response = await fetch(endpoint, {
                method,
                headers,
                body: body ? JSON.stringify(body) : null
            });
             const data = await response.json();
              if (data.error) {
                console.error(`API error (${endpoint}):`, data.message);
                return `Error: ${data.message}`;
            }
           return data;
        } catch (error) {
            console.error(`Network error (${endpoint}):`, error);
            return 'Network error';
        }
    }
           // Authentication
    async login(args) {
        const authCode = args.AUTH_CODE.trim();
        if (!authCode) {
            console.error('Auth code is required');
            return;
        }
               try {
            const response = await fetch('https://oauth.zaloapp.com/v3/access_token', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    app_id: '<APP_ID>', // Replace with your App ID
                    app_secret: '<APP_SECRET>', // Replace with your App Secret
                    code: authCode,
                    redirect_uri: '<REDIRECT_URI>'
                })
            });
                   const data = await response.json();
            if (data.access_token) {
                this.accessToken = data.access_token;
                console.log('Successfully logged in.');
            } else {
                console.error('Login failed:', data);
            }
        } catch (error) {
            console.error('Login error:', error);
        }
    }
     // Get user info
    async getUserInfo() {
        if (this.cache.userInfo) {
            console.log('Returning cached user info.');
            return this.cache.userInfo;
        }
        const endpoint = 'https://openapi.zalo.me/v2.0/me';
        const data = await this.apiCall(endpoint);
        if (data.id && data.name) {
            this.cache.userInfo = `ID: ${data.id}, Name: ${data.name}`;
            return this.cache.userInfo;
        }
        return 'Error retrieving user info';
    }
    // Send text message to user
    async sendMessageToUser(args) {
        const message = args.MESSAGE.trim();
        const userId = args.USER_ID.trim();
        if (!message || !userId) {
            console.error('Message and User ID are required');
            return;
        }
        const endpoint = 'https://openapi.zalo.me/v2.0/oa/message';
        const body = {
            recipient: { user_id: userId },
            message: { text: message }
        };
        const data = await this.apiCall(endpoint, 'POST', body);
        console.log(data);
    }
    // Send image to user
    async sendImageToUser(args) {
        const imageUrl = args.IMAGE_URL.trim();
        const userId = args.USER_ID.trim();
        if (!imageUrl || !userId) {
            console.error('Image URL and User ID are required');
            return;
        }
        const endpoint = 'https://openapi.zalo.me/v2.0/oa/message';
        const body = {
            recipient: { user_id: userId },
            message: {
                attachment: {
                    type: 'image',
                    payload: { url: imageUrl }
                }
            }
        };
        const data = await this.apiCall(endpoint, 'POST', body);
        console.log(data);
    }
    // Send message to group
    async sendMessageToGroup(args) {
        const message = args.MESSAGE.trim();
        const groupId = args.GROUP_ID.trim();
        if (!message || !groupId) {
            console.error('Message and Group ID are required');
            return;
        }
        const endpoint = 'https://openapi.zalo.me/v2.0/oa/group/message';
        const body = {
            group_id: groupId,
            message: { text: message }
        };
        const data = await this.apiCall(endpoint, 'POST', body);
        console.log(data);
    }
    // Get friend list
    async getFriendList() {
        const endpoint = 'https://openapi.zalo.me/v2.0/friend/list';
        const data = await this.apiCall(endpoint);
        if (data && data.friends) {
            this.cache.friends = data.friends;
            return JSON.stringify(data.friends);
        }
        return 'Error retrieving friend list';
    }
    // Send video to user
    async sendVideoToUser(args) {
        const videoUrl = args.VIDEO_URL.trim();
        const userId = args.USER_ID.trim();
        if (!videoUrl || !userId) {
            console.error('Video URL and User ID are required');
            return;
        }
        const endpoint = 'https://openapi.zalo.me/v2.0/oa/message';
        const body = {
            recipient: { user_id: userId },
            message: {
                attachment: {
                    type: 'video',
                    payload: { url: videoUrl }
                }
            }
        };
        const data = await this.apiCall(endpoint, 'POST', body);
        console.log(data);
    }
    // Custom API
    async customApiRequest(args) {
        const endpoint = args.API_ENDPOINT.trim();
        if (!endpoint) {
            console.error('API endpoint is required');
            return;
        }
       const data = await this.apiCall(endpoint);
        return JSON.stringify(data);
    }
}

// Register the extension
Scratch.extensions.register(new ZaloAPI());
