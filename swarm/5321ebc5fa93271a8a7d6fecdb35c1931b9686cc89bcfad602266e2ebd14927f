// SPDX-License-Identifier: MIT
pragma solidity ^0.8.26;

/**
╔════════════════════════════════════════════════════════════════════════════════════════════════════════════╗
║                                                                                                            ║
║  // ==================================== DIGITAL CONTRACT SEAL =========================================   ║
║                                                                                                            ║
║    ██╗  ██╗██╗   ██╗██████╗ ██████╗  ██████╗ ██╗     ███████╗██████╗  ██████╗ ███████╗██████╗              ║
║    ██║  ██║╚██╗ ██╔╝██╔══██╗██╔══██╗██    ██ ██║     ██╔════╝██╔══██╗██╔════╝ ██╔════╝██╔══██╗             ║
║    ███████║ ╚████╔╝ ██   ██╔██████╔╝██    ██ ██║     █████╗  ██║  ██║██║  ███╗█████╗  ██████╔╝             ║
║    ██╔══██║  ╚██╔╝  ██╔══██═██╔══██╗██    ██ ██║     ██╔══╝  ██║  ██║██║   ██║██╔══╝  ██╔══██╗             ║
║    ██║  ██║   ██║   ██║████ ██   ██║██    ██ ██║████ ███████ ███╗███ ███╔╝╚██████╔╝███████╗██║  ██║        ║
║    ╚═╝  ╚═╝   ╚═╝   ╚═╝     ╚═╝  ╚═╝ ██████    ╚══════╝╚══════╝╚═════╝  ╚═════╝ ╚══════╝╚═╝  ╚═╝           ║
║                                                                                                            ║
║  // ======================================= SIGNED & HASHED =============================================  ║
║                                                                                                            ║
╚════════════════════════════════════════════════════════════════════════════════════════════════════════════╝  

 * HydroLedger presents a hybrid RWA-NFT tokenization model that combines the strengths of both fungible and non-fungible tokens
 * for secure, liquid water asset management.
 * DSM (Dynamic Segmentation Mapping) integrated for gas optimization and temporal data management.
 */

import "@openzeppelin/contracts/access/AccessControl.sol";
import "@openzeppelin/contracts/security/ReentrancyGuard.sol";

// DSM Library for Oracle Optimization
library OracleDSM {
    // Temporal segmentation constants
    uint32 constant public ACTIVE_TASK_WINDOW = 1 days;
    uint32 constant public ARCHIVE_THRESHOLD = 30 days;
    uint16 constant public BATCH_SIZE = 25;
    uint16 constant public MAX_ORACLES_PER_TASK = 10;
    
    // Optimized oracle data structure
    struct OptimizedOracle {
        uint32 lastActiveTime;   // 4 bytes
        uint16 completedTasks;   // 2 bytes (max 65,535 tasks)
        uint8 reputationTier;    // 1 byte (0-255 tiers)
        uint8 nodeType;          // 1 byte
        bool isActive;           // 1 byte
        bytes20 ipfsProfileHash; // 20 bytes (truncated for optimization)
    }
    
    // DSM Segment for temporal organization
    struct TemporalSegment {
        uint32 startTime;
        uint32 endTime;
        uint16 activeTasks;
        uint16 completedTasks;
        uint16 activeOracles;
        bool active;
    }
    
    // DSM Classification algorithm for oracle selection
    function classifyOracle(uint32 lastActiveTime, uint16 completedTasks, uint8 reputationTier) 
        internal view returns (uint8) 
    {
        uint32 currentTime = uint32(block.timestamp);
        
        if (lastActiveTime > currentTime - ACTIVE_TASK_WINDOW) {
            return 1; // Recently active - high priority
        } else if (reputationTier >= 200) {
            return 2; // High reputation - premium tier
        } else if (completedTasks > 1000) {
            return 3; // Experienced oracle
        } else if (lastActiveTime < currentTime - ARCHIVE_THRESHOLD) {
            return 4; // Inactive candidate
        }
        return 0; // Standard oracle
    }
    
    // Gas-aware execution decision for oracle tasks
    function shouldProcessImmediate(uint256 gasPrice, uint8 oracleType) internal pure returns (bool) {
        if (gasPrice > 50 gwei && oracleType == 0) {
            return false; // Defer standard oracle tasks during high gas
        }
        return true;
    }
    
    // Optimized oracle selection based on DSM classification
    function selectOptimalOracles(
        address[] memory availableOracles,
        mapping(address => OptimizedOracle) storage oracleData,
        uint8 requiredType
    ) internal view returns (address[] memory) {
        uint256 count;
        address[] memory selected = new address[](MAX_ORACLES_PER_TASK);
        
        for (uint256 i = 0; i < availableOracles.length && count < MAX_ORACLES_PER_TASK; i++) {
            address oracle = availableOracles[i];
            OptimizedOracle storage node = oracleData[oracle];
            
            if (node.isActive && node.nodeType == requiredType) {
                uint8 classification = classifyOracle(node.lastActiveTime, node.completedTasks, node.reputationTier);
                
                if (classification <= 2) { // Only select high-priority oracles
                    selected[count] = oracle;
                    count++;
                }
            }
        }
        
        // Resize array to actual count
        address[] memory finalSelected = new address[](count);
        for (uint256 j = 0; j < count; j++) {
            finalSelected[j] = selected[j];
        }
        
        return finalSelected;
    }
}

contract HydroOracle is AccessControl, ReentrancyGuard {
    using OracleDSM for OracleDSM.OptimizedOracle;
    
    bytes32 public constant DATA_PROVIDER = keccak256("DATA_PROVIDER");
    bytes32 public constant ATTESTATION_PROVIDER = keccak256("ATTESTATION_PROVIDER");
    bytes32 public constant RESERVE_AUDITOR = keccak256("RESERVE_AUDITOR");
    bytes32 public constant COMPUTATION_NODE = keccak256("COMPUTATION_NODE");
    
    enum OracleType { DATA, ATTESTATION, RESERVE }
    
    // DSM Optimized Structures
    struct OptimizedTask {
        uint32 createdAt;        // 4 bytes
        uint32 deadline;         // 4 bytes  
        uint16 requiredType;     // 2 bytes
        uint8 priority;          // 1 byte
        bool completed;          // 1 byte
        address requester;       // 20 bytes
        bytes32 dataSourceHash;  // 32 bytes
        bytes32 computationHash; // 32 bytes
    }
    
    struct OffchainComputation {
        bytes32 computationId;
        address[] selectedOracles;
        uint16[] reputationScores; // Optimized: uint16 instead of uint256
        bytes20 qosProof;          // Optimized: bytes20 instead of bytes32
        bytes20 securityProof;     // Optimized: bytes20 instead of bytes32
        uint32 timestamp;          // Optimized: uint32 instead of uint256
    }
    
    // Core mappings with DSM optimization
    mapping(address => OracleDSM.OptimizedOracle) public oracleNodes;
    mapping(uint256 => OptimizedTask) public tasks;
    mapping(bytes32 => OffchainComputation) public offchainComputations;
    mapping(address => bytes32) public nodePublicKeys;
    
    // DSM Temporal Segments
    mapping(uint8 => OracleDSM.TemporalSegment) public temporalSegments;
    
    // Constants
    uint256 private constant REPUTATION_BIAS = 2;
    uint256 public taskCounter;
    uint8 public currentSegmentId;
    uint32 public lastSegmentUpdate;
    uint32 public totalGasSaved;
    
    // Active oracle lists for quick access
    address[] public activeDataOracles;
    address[] public activeAttestationOracles;
    address[] public activeReserveOracles;
    
    // Events with DSM tracking
    event OracleRegistered(address indexed oracle, OracleType oracleType);
    event TaskCreated(uint256 indexed taskId, OracleType oracleType, bytes32 offchainComputationId);
    event OffchainComputationPosted(bytes32 indexed computationId, address[] selectedOracles);
    event ResponseSubmitted(uint256 indexed taskId, address indexed oracle, bytes response);
    event TaskCompleted(uint256 indexed taskId, bytes finalResponse);
    event ReputationUpdated(address indexed oracle, uint256 newReputation, bytes32 proof);
    event DSMGasOptimized(address indexed user, uint256 gasSaved, uint8 operationType);
    event TemporalSegmentUpdated(uint8 segmentId, uint32 startTime, uint32 endTime);

    constructor() {
        _grantRole(DEFAULT_ADMIN_ROLE, msg.sender);
        
        // Initialize DSM temporal segment
        currentSegmentId = 1;
        lastSegmentUpdate = uint32(block.timestamp);
        temporalSegments[1] = OracleDSM.TemporalSegment({
            startTime: uint32(block.timestamp),
            endTime: uint32(block.timestamp) + OracleDSM.ACTIVE_TASK_WINDOW,
            activeTasks: 0,
            completedTasks: 0,
            activeOracles: 0,
            active: true
        });
    }
    
    // DSM Optimized Oracle Registration
    function registerOracle(
        address oracleAddress,
        OracleType oracleType,
        bytes32 ipfsProfileHash
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        uint256 startGas = gasleft();
        
        require(!oracleNodes[oracleAddress].isActive, "Oracle already registered");
        
        // Convert bytes32 to bytes20 for optimization
        bytes20 optimizedHash = bytes20(ipfsProfileHash);
        
        OracleDSM.OptimizedOracle storage newNode = oracleNodes[oracleAddress];
        newNode.lastActiveTime = uint32(block.timestamp);
        newNode.completedTasks = 0;
        newNode.reputationTier = 50; // Initial tier (0-255)
        newNode.nodeType = uint8(oracleType);
        newNode.isActive = true;
        newNode.ipfsProfileHash = optimizedHash;
        
        // Add to active oracle lists
        if (oracleType == OracleType.DATA) {
            activeDataOracles.push(oracleAddress);
            _grantRole(DATA_PROVIDER, oracleAddress);
        } else if (oracleType == OracleType.ATTESTATION) {
            activeAttestationOracles.push(oracleAddress);
            _grantRole(ATTESTATION_PROVIDER, oracleAddress);
        } else if (oracleType == OracleType.RESERVE) {
            activeReserveOracles.push(oracleAddress);
            _grantRole(RESERVE_AUDITOR, oracleAddress);
        }
        
        // Update temporal segment
        temporalSegments[currentSegmentId].activeOracles++;
        
        // Track gas savings
        uint256 gasUsed = startGas - gasleft();
        uint256 baselineGas = 120000;
        if (baselineGas > gasUsed) {
            totalGasSaved += uint32(baselineGas - gasUsed);
            emit DSMGasOptimized(msg.sender, baselineGas - gasUsed, 1);
        }
        
        emit OracleRegistered(oracleAddress, oracleType);
    }
    
    // DSM Algorithm: Temporal Segment Management
    function updateTemporalSegment(uint32 currentTime) internal {
        OracleDSM.TemporalSegment storage currentSegment = temporalSegments[currentSegmentId];
        
        if (currentTime > currentSegment.endTime) {
            currentSegment.active = false;
            currentSegmentId++;
            
            temporalSegments[currentSegmentId] = OracleDSM.TemporalSegment({
                startTime: currentSegment.endTime + 1,
                endTime: currentSegment.endTime + 1 + OracleDSM.ACTIVE_TASK_WINDOW,
                activeTasks: 0,
                completedTasks: 0,
                activeOracles: uint16(activeDataOracles.length + activeAttestationOracles.length + activeReserveOracles.length),
                active: true
            });
            
            lastSegmentUpdate = currentTime;
            emit TemporalSegmentUpdated(currentSegmentId, 
                temporalSegments[currentSegmentId].startTime, 
                temporalSegments[currentSegmentId].endTime);
        }
    }
    
    // DSM Optimized Task Creation
    function createTask(
        OracleType requiredType,
        string memory dataSource,
        uint256 timeout,
        bytes32 offchainComputationId
    ) external returns (uint256) {
        uint256 startGas = gasleft();
        
        // Update temporal segment
        updateTemporalSegment(uint32(block.timestamp));
        
        uint256 taskId = taskCounter++;
        bytes32 dataSourceHash = keccak256(abi.encodePacked(dataSource));
        
        tasks[taskId] = OptimizedTask({
            createdAt: uint32(block.timestamp),
            deadline: uint32(block.timestamp + timeout),
            requiredType: uint16(requiredType),
            priority: 0, // Will be set based on gas conditions
            completed: false,
            requester: msg.sender,
            dataSourceHash: dataSourceHash,
            computationHash: offchainComputationId
        });
        
        temporalSegments[currentSegmentId].activeTasks++;
        
        // Track gas savings
        uint256 gasUsed = startGas - gasleft();
        uint256 baselineGas = 85000;
        if (baselineGas > gasUsed) {
            totalGasSaved += uint32(baselineGas - gasUsed);
            emit DSMGasOptimized(msg.sender, baselineGas - gasUsed, 2);
        }
        
        emit TaskCreated(taskId, requiredType, offchainComputationId);
        return taskId;
    }
    
    // DSM Optimized Offchain Computation Posting
    function postOffchainComputation(
        bytes32 computationId,
        address[] memory selectedOracles,
        uint256[] memory reputationScores,
        bytes32 qosProof,
        bytes32 securityProof
    ) external onlyRole(COMPUTATION_NODE) {
        uint256 startGas = gasleft();
        
        // Optimize reputation scores storage
        uint16[] memory optimizedScores = new uint16[](reputationScores.length);
        for (uint256 i = 0; i < reputationScores.length; i++) {
            optimizedScores[i] = uint16(reputationScores[i] > 65535 ? 65535 : reputationScores[i]);
        }
        
        offchainComputations[computationId] = OffchainComputation({
            computationId: computationId,
            selectedOracles: selectedOracles,
            reputationScores: optimizedScores,
            qosProof: bytes20(qosProof),
            securityProof: bytes20(securityProof),
            timestamp: uint32(block.timestamp)
        });
        
        // Track gas savings
        uint256 gasUsed = startGas - gasleft();
        uint256 baselineGas = 95000;
        if (baselineGas > gasUsed) {
            totalGasSaved += uint32(baselineGas - gasUsed);
            emit DSMGasOptimized(msg.sender, baselineGas - gasUsed, 3);
        }
        
        emit OffchainComputationPosted(computationId, selectedOracles);
    }
    
    // DSM Optimized Reputation Update
    function updateReputation(
        address oracleAddress,
        uint256 newReputation,
        bytes32 computationProof
    ) external onlyRole(COMPUTATION_NODE) {
        require(oracleNodes[oracleAddress].isActive, "Oracle not active");
        
        oracleNodes[oracleAddress].reputationTier = uint8(newReputation > 255 ? 255 : newReputation);
        oracleNodes[oracleAddress].lastActiveTime = uint32(block.timestamp);
        
        emit ReputationUpdated(oracleAddress, newReputation, computationProof);
    }
    
    // DSM Optimized Response Submission
    function submitResponse(
        uint256 taskId,
        bytes memory response,
        bytes memory signature
    ) external nonReentrant {
        uint256 startGas = gasleft();
        
        require(oracleNodes[msg.sender].isActive, "Oracle not active");
        require(!tasks[taskId].completed, "Task already completed");
        require(block.timestamp <= tasks[taskId].deadline, "Task expired");
        
        // DSM: Gas-aware execution check
        uint8 oracleClass = OracleDSM.classifyOracle(
            oracleNodes[msg.sender].lastActiveTime,
            oracleNodes[msg.sender].completedTasks,
            oracleNodes[msg.sender].reputationTier
        );
        
        require(OracleDSM.shouldProcessImmediate(tx.gasprice, oracleClass), 
            "Deferred due to high gas prices");
        
        // Verify signature
        bytes32 messageHash = keccak256(abi.encodePacked(taskId, response));
        require(_verifySignature(messageHash, signature, msg.sender), "Invalid signature");
        
        // Update oracle metrics
        oracleNodes[msg.sender].completedTasks++;
        oracleNodes[msg.sender].lastActiveTime = uint32(block.timestamp);
        
        // Track gas savings
        uint256 gasUsed = startGas - gasleft();
        uint256 baselineGas = 75000;
        if (baselineGas > gasUsed) {
            totalGasSaved += uint32(baselineGas - gasUsed);
            emit DSMGasOptimized(msg.sender, baselineGas - gasUsed, 4);
        }
        
        emit ResponseSubmitted(taskId, msg.sender, response);
    }
    
    // DSM Optimized Task Completion - FIXED: Commented out unused parameter
    function completeTask(
        uint256 taskId,
        bytes memory finalResponse,
        bytes memory /* aggregationProof */
    ) external onlyRole(COMPUTATION_NODE) {
        require(!tasks[taskId].completed, "Task already completed");
        
        tasks[taskId].completed = true;
        temporalSegments[currentSegmentId].completedTasks++;
        
        emit TaskCompleted(taskId, finalResponse);
    }
    
    // DSM Optimized Oracle Selection
    function getOptimalOraclesForTask(OracleType requiredType) 
        external view returns (address[] memory) 
    {
        address[] memory availableOracles;
        
        if (requiredType == OracleType.DATA) {
            availableOracles = activeDataOracles;
        } else if (requiredType == OracleType.ATTESTATION) {
            availableOracles = activeAttestationOracles;
        } else {
            availableOracles = activeReserveOracles;
        }
        
        return OracleDSM.selectOptimalOracles(availableOracles, oracleNodes, uint8(requiredType));
    }
    
    // DSM Performance Analytics
    function getDSMPerformance() external view returns (
        uint32 gasSaved,
        uint8 activeSegmentId,
        uint16 activeTasks,
        uint16 completedTasks,
        uint16 activeOracles
    ) {
        OracleDSM.TemporalSegment storage segment = temporalSegments[currentSegmentId];
        return (totalGasSaved, currentSegmentId, segment.activeTasks, segment.completedTasks, segment.activeOracles);
    }
    
    // Gas-efficient oracle deactivation
    function deactivateOracle(address oracleAddress) external onlyRole(DEFAULT_ADMIN_ROLE) {
        require(oracleNodes[oracleAddress].isActive, "Oracle not active");
        
        oracleNodes[oracleAddress].isActive = false;
        temporalSegments[currentSegmentId].activeOracles--;
        
        // Remove from active lists (optimized removal)
        _removeFromActiveLists(oracleAddress, OracleType(oracleNodes[oracleAddress].nodeType));
    }
    
    function _removeFromActiveLists(address oracle, OracleType oracleType) internal {
        address[] storage activeList;
        
        if (oracleType == OracleType.DATA) {
            activeList = activeDataOracles;
        } else if (oracleType == OracleType.ATTESTATION) {
            activeList = activeAttestationOracles;
        } else {
            activeList = activeReserveOracles;
        }
        
        for (uint256 i = 0; i < activeList.length; i++) {
            if (activeList[i] == oracle) {
                activeList[i] = activeList[activeList.length - 1];
                activeList.pop();
                break;
            }
        }
    }
    
    // Batch operations for gas efficiency
    function batchRegisterOracles(
        address[] memory oracleAddresses,
        OracleType[] memory oracleTypes,
        bytes32[] memory ipfsProfileHashes
    ) external onlyRole(DEFAULT_ADMIN_ROLE) {
        require(oracleAddresses.length == oracleTypes.length, "Array length mismatch");
        require(oracleAddresses.length == ipfsProfileHashes.length, "Array length mismatch");
        require(oracleAddresses.length <= OracleDSM.BATCH_SIZE, "Batch size exceeded");
        
        uint256 startGas = gasleft();
        
        for (uint256 i = 0; i < oracleAddresses.length; i++) {
            if (!oracleNodes[oracleAddresses[i]].isActive) {
                bytes20 optimizedHash = bytes20(ipfsProfileHashes[i]);
                
                OracleDSM.OptimizedOracle storage newNode = oracleNodes[oracleAddresses[i]];
                newNode.lastActiveTime = uint32(block.timestamp);
                newNode.completedTasks = 0;
                newNode.reputationTier = 50;
                newNode.nodeType = uint8(oracleTypes[i]);
                newNode.isActive = true;
                newNode.ipfsProfileHash = optimizedHash;
                
                // Add to active oracle lists
                if (oracleTypes[i] == OracleType.DATA) {
                    activeDataOracles.push(oracleAddresses[i]);
                    _grantRole(DATA_PROVIDER, oracleAddresses[i]);
                } else if (oracleTypes[i] == OracleType.ATTESTATION) {
                    activeAttestationOracles.push(oracleAddresses[i]);
                    _grantRole(ATTESTATION_PROVIDER, oracleAddresses[i]);
                } else if (oracleTypes[i] == OracleType.RESERVE) {
                    activeReserveOracles.push(oracleAddresses[i]);
                    _grantRole(RESERVE_AUDITOR, oracleAddresses[i]);
                }
                
                temporalSegments[currentSegmentId].activeOracles++;
                emit OracleRegistered(oracleAddresses[i], oracleTypes[i]);
            }
        }
        
        // Track batch gas savings
        uint256 gasUsed = startGas - gasleft();
        uint256 individualGas = oracleAddresses.length * 120000;
        if (individualGas > gasUsed) {
            totalGasSaved += uint32(individualGas - gasUsed);
            emit DSMGasOptimized(msg.sender, individualGas - gasUsed, 5);
        }
    }
    
    // FIXED: Function state mutability restricted to pure
    function _verifySignature(
        bytes32 messageHash,
        bytes memory signature,
        address expectedSigner
    ) internal pure returns (bool) {
        bytes32 ethSignedMessageHash = keccak256(
            abi.encodePacked("\x19Ethereum Signed Message:\n32", messageHash)
        );
        address recovered = _recoverSigner(ethSignedMessageHash, signature);
        return recovered == expectedSigner;
    }
    
    function _recoverSigner(bytes32 messageHash, bytes memory signature) internal pure returns (address) {
        (bytes32 r, bytes32 s, uint8 v) = _splitSignature(signature);
        return ecrecover(messageHash, v, r, s);
    }
    
    function _splitSignature(bytes memory signature) internal pure returns (bytes32 r, bytes32 s, uint8 v) {
        require(signature.length == 65, "Invalid signature length");
        assembly {
            r := mload(add(signature, 32))
            s := mload(add(signature, 64))
            v := byte(0, mload(add(signature, 96)))
        }
        if (v < 27) v += 27;
    }
    
    // View functions for frontend integration
    function getActiveOraclesCount() external view returns (
        uint256 dataOracles,
        uint256 attestationOracles,
        uint256 reserveOracles
    ) {
        return (
            activeDataOracles.length,
            activeAttestationOracles.length,
            activeReserveOracles.length
        );
    }
    
    function getOracleInfo(address oracleAddress) external view returns (
        uint32 lastActiveTime,
        uint16 completedTasks,
        uint8 reputationTier,
        uint8 nodeType,
        bool isActive,
        bytes20 ipfsProfileHash
    ) {
        OracleDSM.OptimizedOracle storage oracle = oracleNodes[oracleAddress];
        return (
            oracle.lastActiveTime,
            oracle.completedTasks,
            oracle.reputationTier,
            oracle.nodeType,
            oracle.isActive,
            oracle.ipfsProfileHash
        );
    }
    
    // Additional utility functions
    function getTaskInfo(uint256 taskId) external view returns (
        uint32 createdAt,
        uint32 deadline,
        uint16 requiredType,
        uint8 priority,
        bool completed,
        address requester,
        bytes32 dataSourceHash,
        bytes32 computationHash
    ) {
        OptimizedTask storage task = tasks[taskId];
        return (
            task.createdAt,
            task.deadline,
            task.requiredType,
            task.priority,
            task.completed,
            task.requester,
            task.dataSourceHash,
            task.computationHash
        );
    }
    
    function getCurrentTemporalSegment() external view returns (
        uint8 segmentId,
        uint32 startTime,
        uint32 endTime,
        uint16 activeTasks,
        uint16 completedTasks,
        uint16 activeOracles,
        bool active
    ) {
        OracleDSM.TemporalSegment storage segment = temporalSegments[currentSegmentId];
        return (
            currentSegmentId,
            segment.startTime,
            segment.endTime,
            segment.activeTasks,
            segment.completedTasks,
            segment.activeOracles,
            segment.active
        );
    }
}