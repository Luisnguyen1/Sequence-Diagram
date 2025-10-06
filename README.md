sequenceDiagram
    actor Employee as Nhân viên
    participant UI as Dashboard/UI
    participant HrEmployee as hr.employee
    participant HrAttendance as hr.attendance
    participant Camera as Camera/Hanet
    participant AttendanceSummary as hr.attendance.summary
    participant Manager as Quản lý

    Note over Employee,Manager: CHẤM CÔNG THƯỜNG NGÀY
    
    Employee->>UI: Mở Dashboard
    UI->>HrEmployee: get_user_employee_details()
    HrEmployee-->>UI: Hiển thị thông tin chấm công
    
    alt Chấm công bằng Camera
        Camera->>HrAttendance: Ghi nhận check-in tự động
        HrAttendance->>HrAttendance: Lưu camera_id, hanet_chekintime
        HrAttendance->>HrAttendance: state = 'draft'
    else Chấm công thủ công
        Employee->>UI: Click "Chấm công"
        UI->>HrEmployee: _attendance_action_change()
        HrEmployee->>HrEmployee: Kiểm tra quyền
        alt Không có quyền manual
            HrEmployee-->>Employee: UserError: "Vui lòng chấm công bằng camera"
        else Có quyền
            HrEmployee->>HrAttendance: create() - Check In
            HrAttendance->>HrAttendance: Lưu thông tin geo_information
        end
    end
    
    Note over Employee,Manager: TÍNH CÔNG THÁNG
    
    Manager->>UI: Truy cập Cấu hình tính công
    UI->>HrConfigAttendancePayslip: Cấu hình ngày chốt công
    HrConfigAttendancePayslip->>HrConfigAttendancePayslip: mv_ngay_chot_cong_p11/p12
    
    Note over AttendanceSummary: Cron Job tự động
    AttendanceSummary->>HrEmployee: compute_data_month_report()
    HrEmployee->>HrAttendance: Lấy dữ liệu chấm công theo tháng
    HrEmployee->>AttendanceSummary: Tính toán công chi tiết
    AttendanceSummary->>AttendanceSummary: _compute_detail_working_days()
    AttendanceSummary->>AttendanceSummary: compute_leave_type_payslip()
    
    Manager->>AttendanceSummary: Xem bảng công
    AttendanceSummary-->>Manager: Hiển thị chi tiết
    
    alt Nhân viên xác nhận
        Employee->>AttendanceSummary: action_employee_approve()
        AttendanceSummary->>AttendanceSummary: mv_confirm_attendance_employee = 'approve'
    else Yêu cầu xác nhận lại
        Employee->>AttendanceSummary: Gửi yêu cầu
        AttendanceSummary->>AttendanceSummary: mv_confirm_attendance_employee = 'ask_agian'
    end
    
    Manager->>AttendanceSummary: action_approve()
    AttendanceSummary->>AttendanceSummary: mv_state = 'approved'
    AttendanceSummary->>AttendanceSummary: mv_gen_payslip_done = True
