// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract SocialNetwork {
    // 用户结构
    struct User {
        address id;
        string name;
        uint reputation;
    }

    // 消息结构
    struct Message {
        uint id;
        address author;
        string content;
        uint timestamp;
        uint likesCount;
    }

    // 评论结构
    struct Comment {
        uint id;
        uint messageId;
        address author;
        string content;
        uint timestamp;
    }

    // 用户列表
    mapping(address => User) public users;
    // 消息列表
    mapping(uint => Message) public messages;
    // 评论列表
    mapping(uint => Comment) public comments;
    // 用户对消息的点赞记录
    mapping(address => mapping(uint => bool)) public likes;

    // 计数器
    uint public userCount;
    uint public messageCount;
    uint public commentCount;

    // 事件
    event UserRegistered(address indexed id, string name);
    event NewMessage(uint indexed id, address indexed author, string content, uint timestamp);
    event MessageLiked(uint indexed messageId, address indexed liker);
    event NewComment(uint indexed id, uint indexed messageId, address indexed author, string content, uint timestamp);

    // 注册用户
    function registerUser(string memory _name) public {
        require(bytes(users[msg.sender].name).length == 0, "User already registered");
        userCount++;
        users[msg.sender] = User(msg.sender, _name, 0);
        emit UserRegistered(msg.sender, _name);
    }

    // 发布消息
    function createMessage(string memory _content) public {
        require(bytes(users[msg.sender].name).length != 0, "User not registered");
        messageCount++;
        messages[messageCount] = Message(messageCount, msg.sender, _content, block.timestamp, 0);
        emit NewMessage(messageCount, msg.sender, _content, block.timestamp);
    }

    // 点赞消息
    function likeMessage(uint _messageId) public {
        require(_messageId > 0 && _messageId <= messageCount, "Invalid message ID");
        require(!likes[msg.sender][_messageId], "Already liked this message");
        likes[msg.sender][_messageId] = true;
        messages[_messageId].likesCount++;
        emit MessageLiked(_messageId, msg.sender);
    }

    // 添加评论
    function addComment(uint _messageId, string memory _content) public {
        require(_messageId > 0 && _messageId <= messageCount, "Invalid message ID");
        require(bytes(users[msg.sender].name).length != 0, "User not registered");
        commentCount++;
        comments[commentCount] = Comment(commentCount, _messageId, msg.sender, _content, block.timestamp);
        emit NewComment(commentCount, _messageId, msg.sender, _content, block.timestamp);
    }

    // 获取所有消息
    function getAllMessages() public view returns (Message[] memory) {
        Message[] memory allMessages = new Message[](messageCount);
        for (uint i = 1; i <= messageCount; i++) {
            allMessages[i - 1] = messages[i];
        }
        return allMessages;
    }

    // 获取特定消息的评论
    function getCommentsForMessage(uint _messageId) public view returns (Comment[] memory) {
        uint count = 0;
        for (uint i = 1; i <= commentCount; i++) {
            if (comments[i].messageId == _messageId) {
                count++;
            }
        }

        Comment[] memory result = new Comment[](count);
        uint counter = 0;
        for (uint i = 1; i <= commentCount; i++) {
            if (comments[i].messageId == _messageId) {
                result[counter] = comments[i];
                counter++;
            }
        }
        return result;
    }

    // 获取用户信息
    function getUser(address _id) public view returns (User memory) {
        return users[_id];
    }
}
