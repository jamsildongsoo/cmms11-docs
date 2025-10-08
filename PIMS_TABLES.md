# 구매관리시스템 데이터베이스 설계 (Purchase Management System Tables)

## 1. 데이터베이스 개요

### 1.1 핵심 테이블 관계도
```
vendor_master (협력사)
    ├── vendor_category (분류)
    ├── vendor_evaluation (평가)
    └── bidding_participant (입찰참가)

purchase_request (구매요청)
    ├── purchase_request_item (상세)
    ├── purchase_request_approval (결재)
    └── bidding_notice (입찰공고)

bidding_notice (입찰공고)
    ├── bidding_participant (참가업체)
    └── bidding_proposal (입찰서)

bidding_proposal (입찰서)
    ├── bidding_proposal_item (상세)
    └── bidding_evaluation (평가)

purchase_order (구매오더)
    ├── purchase_order_item (상세)
    └── purchase_order_history (이력)

receiving_header (입고)
    ├── receiving_item (상세)
    └── inventory_transaction (재고이동)
```

### 1.2 주요 인덱스 설계
```sql
-- 성능 최적화를 위한 인덱스
CREATE INDEX idx_vendor_status ON vendor_master(status);
CREATE INDEX idx_request_date ON purchase_request(request_date);
CREATE INDEX idx_request_status ON purchase_request(status);
CREATE INDEX idx_bidding_date ON bidding_notice(bid_start_date, bid_end_date);
CREATE INDEX idx_po_vendor ON purchase_order(vendor_id);
CREATE INDEX idx_receiving_date ON receiving_header(receiving_date);
```

## 2. 협력사 관리 테이블

### 2.1 협력사 마스터
```sql
-- 협력사 마스터
CREATE TABLE vendor_master (
    vendor_id VARCHAR(10) PRIMARY KEY,
    vendor_name VARCHAR(100) NOT NULL,
    vendor_type VARCHAR(20), -- 'COMPANY', 'INDIVIDUAL'
    business_no VARCHAR(20), -- 사업자등록번호
    ceo_name VARCHAR(50),
    address VARCHAR(200),
    phone VARCHAR(20),
    email VARCHAR(100),
    contact_person VARCHAR(50),
    contact_phone VARCHAR(20),
    contact_email VARCHAR(100),
    bank_name VARCHAR(50),
    bank_account VARCHAR(50),
    account_holder VARCHAR(50),
    status VARCHAR(20) DEFAULT 'PENDING', -- 'PENDING', 'APPROVED', 'REJECTED', 'SUSPENDED'
    approval_date DATETIME,
    approved_by VARCHAR(20),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20),
    updated_at DATETIME,
    updated_by VARCHAR(20),
    delete_mark CHAR(1) DEFAULT 'N'
);
```

### 2.2 협력사 분류
```sql
-- 협력사 분류
CREATE TABLE vendor_category (
    vendor_id VARCHAR(10),
    category_code VARCHAR(20), -- 'ELECTRONIC', 'MECHANICAL', 'CHEMICAL', etc.
    PRIMARY KEY (vendor_id, category_code),
    FOREIGN KEY (vendor_id) REFERENCES vendor_master(vendor_id)
);
```

### 2.3 협력사 평가
```sql
-- 협력사 평가
CREATE TABLE vendor_evaluation (
    evaluation_id VARCHAR(20) PRIMARY KEY,
    vendor_id VARCHAR(10),
    evaluation_date DATE,
    quality_score DECIMAL(3,1),
    delivery_score DECIMAL(3,1),
    service_score DECIMAL(3,1),
    price_score DECIMAL(3,1),
    total_score DECIMAL(3,1),
    evaluation_comment TEXT,
    evaluator VARCHAR(20),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (vendor_id) REFERENCES vendor_master(vendor_id)
);
```

## 3. 구매요청 관리 테이블

### 3.1 구매요청 헤더
```sql
-- 구매요청 헤더
CREATE TABLE purchase_request (
    request_id VARCHAR(20) PRIMARY KEY,
    request_date DATE NOT NULL,
    request_dept VARCHAR(20),
    requester VARCHAR(20),
    purpose VARCHAR(200),
    required_date DATE,
    status VARCHAR(20) DEFAULT 'DRAFT', -- 'DRAFT', 'SUBMITTED', 'APPROVED', 'REJECTED'
    approval_status VARCHAR(20), -- 'PENDING', 'APPROVED', 'REJECTED'
    total_amount DECIMAL(15,2),
    currency VARCHAR(3) DEFAULT 'KRW',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20),
    updated_at DATETIME,
    updated_by VARCHAR(20)
);
```

### 3.2 구매요청 상세
```sql
-- 구매요청 상세
CREATE TABLE purchase_request_item (
    request_id VARCHAR(20),
    item_seq INT,
    inventory_id VARCHAR(20), -- CMMS11 inventory 연계
    item_name VARCHAR(200),
    specification TEXT,
    quantity DECIMAL(10,2),
    unit VARCHAR(20),
    unit_price DECIMAL(15,2),
    total_price DECIMAL(15,2),
    required_date DATE,
    remark TEXT,
    PRIMARY KEY (request_id, item_seq),
    FOREIGN KEY (request_id) REFERENCES purchase_request(request_id)
);
```

### 3.3 구매요청 결재
```sql
-- 구매요청 결재
CREATE TABLE purchase_request_approval (
    request_id VARCHAR(20),
    approval_seq INT,
    approver VARCHAR(20),
    approval_type VARCHAR(20), -- 'APPROVAL', 'AGREE', 'INFORM'
    approval_status VARCHAR(20) DEFAULT 'PENDING',
    approval_date DATETIME,
    approval_comment TEXT,
    PRIMARY KEY (request_id, approval_seq),
    FOREIGN KEY (request_id) REFERENCES purchase_request(request_id)
);
```

## 4. 입찰 관리 테이블

### 4.1 입찰 공고
```sql
-- 입찰 공고
CREATE TABLE bidding_notice (
    notice_id VARCHAR(20) PRIMARY KEY,
    notice_title VARCHAR(200),
    notice_type VARCHAR(20), -- 'OPEN', 'LIMITED', 'NEGOTIATION'
    request_id VARCHAR(20),
    notice_date DATE,
    bid_start_date DATETIME,
    bid_end_date DATETIME,
    bid_open_date DATETIME,
    status VARCHAR(20) DEFAULT 'DRAFT', -- 'DRAFT', 'PUBLISHED', 'CLOSED', 'CANCELLED'
    description TEXT,
    terms_conditions TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20),
    FOREIGN KEY (request_id) REFERENCES purchase_request(request_id)
);
```

### 4.2 입찰 참가업체
```sql
-- 입찰 참가업체
CREATE TABLE bidding_participant (
    notice_id VARCHAR(20),
    vendor_id VARCHAR(10),
    participation_date DATETIME,
    status VARCHAR(20) DEFAULT 'REGISTERED', -- 'REGISTERED', 'WITHDRAWN', 'DISQUALIFIED'
    PRIMARY KEY (notice_id, vendor_id),
    FOREIGN KEY (notice_id) REFERENCES bidding_notice(notice_id),
    FOREIGN KEY (vendor_id) REFERENCES vendor_master(vendor_id)
);
```

### 4.3 입찰서
```sql
-- 입찰서
CREATE TABLE bidding_proposal (
    proposal_id VARCHAR(20) PRIMARY KEY,
    notice_id VARCHAR(20),
    vendor_id VARCHAR(10),
    proposal_date DATETIME,
    total_amount DECIMAL(15,2),
    currency VARCHAR(3),
    delivery_period INT, -- 일수
    validity_period INT, -- 일수
    payment_terms VARCHAR(100),
    proposal_file VARCHAR(200),
    status VARCHAR(20) DEFAULT 'SUBMITTED', -- 'SUBMITTED', 'EVALUATED', 'SELECTED', 'REJECTED'
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (notice_id) REFERENCES bidding_notice(notice_id),
    FOREIGN KEY (vendor_id) REFERENCES vendor_master(vendor_id)
);
```

### 4.4 입찰서 상세
```sql
-- 입찰서 상세
CREATE TABLE bidding_proposal_item (
    proposal_id VARCHAR(20),
    item_seq INT,
    inventory_id VARCHAR(20),
    item_name VARCHAR(200),
    specification TEXT,
    quantity DECIMAL(10,2),
    unit VARCHAR(20),
    unit_price DECIMAL(15,2),
    total_price DECIMAL(15,2),
    delivery_period INT,
    remark TEXT,
    PRIMARY KEY (proposal_id, item_seq),
    FOREIGN KEY (proposal_id) REFERENCES bidding_proposal(proposal_id)
);
```

### 4.5 입찰 평가
```sql
-- 입찰 평가
CREATE TABLE bidding_evaluation (
    evaluation_id VARCHAR(20) PRIMARY KEY,
    notice_id VARCHAR(20),
    proposal_id VARCHAR(20),
    evaluator VARCHAR(20),
    technical_score DECIMAL(5,2),
    price_score DECIMAL(5,2),
    delivery_score DECIMAL(5,2),
    total_score DECIMAL(5,2),
    evaluation_comment TEXT,
    evaluation_date DATETIME,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (notice_id) REFERENCES bidding_notice(notice_id),
    FOREIGN KEY (proposal_id) REFERENCES bidding_proposal(proposal_id)
);
```

## 5. 구매오더 관리 테이블

### 5.1 구매오더 헤더
```sql
-- 구매오더 헤더
CREATE TABLE purchase_order (
    po_id VARCHAR(20) PRIMARY KEY,
    po_date DATE NOT NULL,
    vendor_id VARCHAR(10),
    notice_id VARCHAR(20),
    proposal_id VARCHAR(20),
    request_id VARCHAR(20),
    po_type VARCHAR(20), -- 'PURCHASE', 'SERVICE', 'CONTRACT'
    status VARCHAR(20) DEFAULT 'ISSUED', -- 'ISSUED', 'CONFIRMED', 'SHIPPED', 'RECEIVED', 'CLOSED'
    total_amount DECIMAL(15,2),
    currency VARCHAR(3),
    payment_terms VARCHAR(100),
    delivery_address VARCHAR(200),
    delivery_date DATE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20),
    FOREIGN KEY (vendor_id) REFERENCES vendor_master(vendor_id),
    FOREIGN KEY (notice_id) REFERENCES bidding_notice(notice_id),
    FOREIGN KEY (proposal_id) REFERENCES bidding_proposal(proposal_id),
    FOREIGN KEY (request_id) REFERENCES purchase_request(request_id)
);
```

### 5.2 구매오더 상세
```sql
-- 구매오더 상세
CREATE TABLE purchase_order_item (
    po_id VARCHAR(20),
    item_seq INT,
    inventory_id VARCHAR(20),
    item_name VARCHAR(200),
    specification TEXT,
    quantity DECIMAL(10,2),
    unit VARCHAR(20),
    unit_price DECIMAL(15,2),
    total_price DECIMAL(15,2),
    delivery_date DATE,
    received_quantity DECIMAL(10,2) DEFAULT 0,
    remark TEXT,
    PRIMARY KEY (po_id, item_seq),
    FOREIGN KEY (po_id) REFERENCES purchase_order(po_id)
);
```

### 5.3 구매오더 이력
```sql
-- 구매오더 이력
CREATE TABLE purchase_order_history (
    history_id VARCHAR(20) PRIMARY KEY,
    po_id VARCHAR(20),
    status VARCHAR(20),
    status_date DATETIME,
    remark TEXT,
    created_by VARCHAR(20),
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (po_id) REFERENCES purchase_order(po_id)
);
```

## 6. 검수/입고 관리 테이블

### 6.1 입고 헤더
```sql
-- 입고 헤더
CREATE TABLE receiving_header (
    receiving_id VARCHAR(20) PRIMARY KEY,
    receiving_date DATE NOT NULL,
    po_id VARCHAR(20),
    vendor_id VARCHAR(10),
    receiving_type VARCHAR(20), -- 'FULL', 'PARTIAL', 'OVER'
    status VARCHAR(20) DEFAULT 'RECEIVED', -- 'RECEIVED', 'INSPECTED', 'APPROVED', 'REJECTED'
    total_amount DECIMAL(15,2),
    inspector VARCHAR(20),
    inspection_date DATETIME,
    inspection_result VARCHAR(20), -- 'PASS', 'FAIL', 'CONDITIONAL'
    inspection_comment TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20),
    FOREIGN KEY (po_id) REFERENCES purchase_order(po_id),
    FOREIGN KEY (vendor_id) REFERENCES vendor_master(vendor_id)
);
```

### 6.2 입고 상세
```sql
-- 입고 상세
CREATE TABLE receiving_item (
    receiving_id VARCHAR(20),
    item_seq INT,
    po_id VARCHAR(20),
    po_item_seq INT,
    inventory_id VARCHAR(20),
    item_name VARCHAR(200),
    ordered_quantity DECIMAL(10,2),
    received_quantity DECIMAL(10,2),
    accepted_quantity DECIMAL(10,2),
    rejected_quantity DECIMAL(10,2),
    unit VARCHAR(20),
    unit_price DECIMAL(15,2),
    total_price DECIMAL(15,2),
    inspection_result VARCHAR(20),
    inspection_comment TEXT,
    storage_location VARCHAR(50),
    PRIMARY KEY (receiving_id, item_seq),
    FOREIGN KEY (receiving_id) REFERENCES receiving_header(receiving_id),
    FOREIGN KEY (po_id) REFERENCES purchase_order(po_id)
);
```

### 6.3 재고 이동 (CMMS11 inventoryTx 연계)
```sql
-- 재고 이동 (CMMS11 inventoryTx 연계)
CREATE TABLE inventory_transaction (
    transaction_id VARCHAR(20) PRIMARY KEY,
    transaction_date DATE NOT NULL,
    transaction_type VARCHAR(20), -- 'PURCHASE_RECEIPT', 'PURCHASE_RETURN'
    receiving_id VARCHAR(20),
    inventory_id VARCHAR(20),
    quantity DECIMAL(10,2),
    unit_cost DECIMAL(15,2),
    total_cost DECIMAL(15,2),
    storage_location VARCHAR(50),
    remark TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20),
    FOREIGN KEY (receiving_id) REFERENCES receiving_header(receiving_id)
);
```

## 7. 시스템 연계 테이블

### 7.1 CMMS11 연계 테이블
```sql
-- CMMS11 Inventory 연계 테이블
CREATE TABLE cmms_inventory_sync (
    sync_id VARCHAR(20) PRIMARY KEY,
    inventory_id VARCHAR(20),
    sync_type VARCHAR(20), -- 'CREATE', 'UPDATE', 'DELETE'
    sync_status VARCHAR(20), -- 'PENDING', 'SUCCESS', 'FAILED'
    sync_date DATETIME,
    error_message TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);

-- CMMS11 InventoryTx 연계 테이블
CREATE TABLE cmms_inventory_tx_sync (
    sync_id VARCHAR(20) PRIMARY KEY,
    transaction_id VARCHAR(20),
    sync_type VARCHAR(20), -- 'CREATE', 'UPDATE', 'DELETE'
    sync_status VARCHAR(20), -- 'PENDING', 'SUCCESS', 'FAILED'
    sync_date DATETIME,
    error_message TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

### 7.2 ERP 연계 테이블
```sql
-- ERP 회계전표 연계 테이블
CREATE TABLE erp_voucher_sync (
    sync_id VARCHAR(20) PRIMARY KEY,
    po_id VARCHAR(20),
    voucher_type VARCHAR(20), -- 'PURCHASE', 'PAYMENT', 'RECEIPT'
    sync_status VARCHAR(20), -- 'PENDING', 'SUCCESS', 'FAILED'
    sync_date DATETIME,
    error_message TEXT,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (po_id) REFERENCES purchase_order(po_id)
);
```

## 8. 시스템 관리 테이블

### 8.1 코드 관리
```sql
-- 코드 타입
CREATE TABLE code_type (
    code_type VARCHAR(20) PRIMARY KEY,
    code_name VARCHAR(100),
    description TEXT,
    use_yn CHAR(1) DEFAULT 'Y',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20)
);

-- 코드 아이템
CREATE TABLE code_item (
    code_type VARCHAR(20),
    code_value VARCHAR(20),
    code_name VARCHAR(100),
    sort_order INT,
    use_yn CHAR(1) DEFAULT 'Y',
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20),
    PRIMARY KEY (code_type, code_value),
    FOREIGN KEY (code_type) REFERENCES code_type(code_type)
);
```

### 8.2 파일 관리
```sql
-- 파일 그룹
CREATE TABLE file_group (
    file_group_id VARCHAR(20) PRIMARY KEY,
    group_name VARCHAR(100),
    group_type VARCHAR(20), -- 'VENDOR', 'REQUEST', 'BIDDING', 'PO', 'RECEIVING'
    reference_id VARCHAR(20), -- 관련 테이블의 ID
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    created_by VARCHAR(20)
);

-- 파일 아이템
CREATE TABLE file_item (
    file_id VARCHAR(20) PRIMARY KEY,
    file_group_id VARCHAR(20),
    file_name VARCHAR(200),
    original_name VARCHAR(200),
    file_path VARCHAR(500),
    file_size BIGINT,
    file_type VARCHAR(50),
    upload_date DATETIME DEFAULT CURRENT_TIMESTAMP,
    upload_by VARCHAR(20),
    FOREIGN KEY (file_group_id) REFERENCES file_group(file_group_id)
);
```

## 9. 데이터베이스 제약조건

### 9.1 체크 제약조건
```sql
-- 상태값 체크
ALTER TABLE vendor_master ADD CONSTRAINT chk_vendor_status 
    CHECK (status IN ('PENDING', 'APPROVED', 'REJECTED', 'SUSPENDED'));

ALTER TABLE purchase_request ADD CONSTRAINT chk_request_status 
    CHECK (status IN ('DRAFT', 'SUBMITTED', 'APPROVED', 'REJECTED'));

ALTER TABLE bidding_notice ADD CONSTRAINT chk_notice_status 
    CHECK (status IN ('DRAFT', 'PUBLISHED', 'CLOSED', 'CANCELLED'));

ALTER TABLE purchase_order ADD CONSTRAINT chk_po_status 
    CHECK (status IN ('ISSUED', 'CONFIRMED', 'SHIPPED', 'RECEIVED', 'CLOSED'));

ALTER TABLE receiving_header ADD CONSTRAINT chk_receiving_status 
    CHECK (status IN ('RECEIVED', 'INSPECTED', 'APPROVED', 'REJECTED'));
```

### 9.2 데이터 타입 제약조건
```sql
-- 금액 필드 체크
ALTER TABLE purchase_request ADD CONSTRAINT chk_total_amount 
    CHECK (total_amount >= 0);

ALTER TABLE purchase_order ADD CONSTRAINT chk_po_amount 
    CHECK (total_amount >= 0);

ALTER TABLE receiving_header ADD CONSTRAINT chk_receiving_amount 
    CHECK (total_amount >= 0);

-- 수량 필드 체크
ALTER TABLE purchase_request_item ADD CONSTRAINT chk_quantity 
    CHECK (quantity > 0);

ALTER TABLE purchase_order_item ADD CONSTRAINT chk_po_quantity 
    CHECK (quantity > 0);

ALTER TABLE receiving_item ADD CONSTRAINT chk_received_quantity 
    CHECK (received_quantity >= 0);
```

## 10. 성능 최적화

### 10.1 파티셔닝
```sql
-- 날짜별 파티셔닝 (대용량 데이터 처리)
ALTER TABLE purchase_request PARTITION BY RANGE (YEAR(request_date)) (
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p2026 VALUES LESS THAN (2027),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);

ALTER TABLE receiving_header PARTITION BY RANGE (YEAR(receiving_date)) (
    PARTITION p2024 VALUES LESS THAN (2025),
    PARTITION p2025 VALUES LESS THAN (2026),
    PARTITION p2026 VALUES LESS THAN (2027),
    PARTITION p_future VALUES LESS THAN MAXVALUE
);
```

### 10.2 뷰 생성
```sql
-- 구매 현황 뷰
CREATE VIEW v_purchase_status AS
SELECT 
    pr.request_id,
    pr.request_date,
    pr.request_dept,
    pr.requester,
    pr.status,
    pr.total_amount,
    po.po_id,
    po.po_date,
    po.status as po_status,
    rh.receiving_id,
    rh.receiving_date,
    rh.status as receiving_status
FROM purchase_request pr
LEFT JOIN purchase_order po ON pr.request_id = po.request_id
LEFT JOIN receiving_header rh ON po.po_id = rh.po_id;

-- 협력사 거래 현황 뷰
CREATE VIEW v_vendor_transaction AS
SELECT 
    vm.vendor_id,
    vm.vendor_name,
    vm.status,
    COUNT(DISTINCT po.po_id) as po_count,
    SUM(po.total_amount) as total_amount,
    COUNT(DISTINCT rh.receiving_id) as receiving_count,
    AVG(ve.total_score) as avg_evaluation_score
FROM vendor_master vm
LEFT JOIN purchase_order po ON vm.vendor_id = po.vendor_id
LEFT JOIN receiving_header rh ON po.po_id = rh.po_id
LEFT JOIN vendor_evaluation ve ON vm.vendor_id = ve.vendor_id
GROUP BY vm.vendor_id, vm.vendor_name, vm.status;
```
