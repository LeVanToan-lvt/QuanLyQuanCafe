# Tìm kiếm
# TimKiem.aspx
<%@ Page Language="C#" AutoEventWireup="true" CodeFile="TimKiem.aspx.cs" Inherits="TimKiem"
    MasterPageFile="~/Site1.Master" %>

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <div class="container" style="max-width:900px; margin:auto; padding:20px;">
        <h2 style="text-align:center; margin-bottom:20px;">Tìm kiếm món / đồ uống</h2>

        <!-- Khung tìm kiếm -->
        <asp:Panel ID="pnlSearch" runat="server" CssClass="card" 
            Style="padding:15px; margin-bottom:20px; box-shadow:0 0 8px #ccc; border-radius:10px;">
            <div style="display:flex; flex-wrap:wrap; gap:10px; align-items:flex-end;">

                <!-- Từ khóa -->
                <div style="flex:2; min-width:220px;">
                    <asp:Label ID="lblTuKhoa" runat="server" Text="Từ khóa:" AssociatedControlID="txtTuKhoa"></asp:Label>
                    <asp:TextBox ID="txtTuKhoa" runat="server" CssClass="form-control" 
                        placeholder="Nhập tên món, mã món, loại món..."></asp:TextBox>
                </div>

                <!-- Loại tìm kiếm -->
                <div style="flex:1; min-width:180px;">
                    <asp:Label ID="lblLoai" runat="server" Text="Loại:" AssociatedControlID="ddlLoai"></asp:Label>
                    <asp:DropDownList ID="ddlLoai" runat="server" CssClass="form-select">
                        <asp:ListItem Text="Tất cả" Value=""></asp:ListItem>
                        <asp:ListItem Text="Cà phê" Value="Cafe"></asp:ListItem>
                        <asp:ListItem Text="Trà" Value="Tra"></asp:ListItem>
                        <asp:ListItem Text="Sinh tố" Value="SinhTo"></asp:ListItem>
                        <asp:ListItem Text="Bánh ngọt" Value="Banh"></asp:ListItem>
                    </asp:DropDownList>
                </div>

                <!-- Sắp xếp -->
                <div style="flex:1; min-width:180px;">
                    <asp:Label ID="lblSort" runat="server" Text="Sắp xếp:" AssociatedControlID="ddlSort"></asp:Label>
                    <asp:DropDownList ID="ddlSort" runat="server" CssClass="form-select">
                        <asp:ListItem Text="Tên A → Z" Value="NameAsc"></asp:ListItem>
                        <asp:ListItem Text="Tên Z → A" Value="NameDesc"></asp:ListItem>
                        <asp:ListItem Text="Giá tăng dần" Value="PriceAsc"></asp:ListItem>
                        <asp:ListItem Text="Giá giảm dần" Value="PriceDesc"></asp:ListItem>
                    </asp:DropDownList>
                </div>

                <!-- Nút tìm kiếm -->
                <div style="flex:0; min-width:120px; text-align:right;">
                    <asp:Button ID="btnTimKiem" runat="server" Text="Tìm kiếm" CssClass="btn btn-primary"
                        OnClick="btnTimKiem_Click" />
                </div>
            </div>
        </asp:Panel>

        <!-- Kết quả -->
        <asp:Panel ID="pnlResult" runat="server">
            <asp:Label ID="lblThongBao" runat="server" ForeColor="Red"></asp:Label>

            <asp:GridView ID="gvKetQua" runat="server" AutoGenerateColumns="False"
                CssClass="table table-striped table-bordered"
                EmptyDataText="Không tìm thấy món phù hợp.">
                <Columns>
                    <asp:BoundField DataField="MaMon" HeaderText="Mã món" />
                    <asp:BoundField DataField="TenMon" HeaderText="Tên món" />
                    <asp:BoundField DataField="Loai" HeaderText="Loại" />
                    <asp:BoundField DataField="Gia" HeaderText="Giá (VND)" DataFormatString="{0:N0}" />
                    <asp:BoundField DataField="TrangThai" HeaderText="Trạng thái" />
                    <asp:HyperLinkField HeaderText="Xem chi tiết" Text="Chi tiết"
                        DataNavigateUrlFields="MaMon"
                        DataNavigateUrlFormatString="DanhGiaBinhLuan.aspx?maMon={0}" />
                </Columns>
            </asp:GridView>
        </asp:Panel>

#TimKiem.aspx.cs
using System;
using System.Data;

public partial class TimKiem : System.Web.UI.Page
{
    protected void Page_Load(object sender, EventArgs e)
    {
        if (!IsPostBack)
        {
            BindDummyData();
        }
    }

    private DataTable TaoBangMau()
    {
        DataTable dt = new DataTable();
        dt.Columns.Add("MaMon");
        dt.Columns.Add("TenMon");
        dt.Columns.Add("Loai");
        dt.Columns.Add("Gia", typeof(int));
        dt.Columns.Add("TrangThai");

        dt.Rows.Add("CF01", "Cà phê đen đá", "Cafe", 25000, "Đang bán");
        dt.Rows.Add("CF02", "Cà phê sữa đá", "Cafe", 30000, "Đang bán");
        dt.Rows.Add("TR01", "Trà đào cam sả", "Tra", 35000, "Đang bán");
        dt.Rows.Add("ST01", "Sinh tố bơ", "SinhTo", 40000, "Đang bán");
        dt.Rows.Add("BK01", "Bánh tiramisu", "Banh", 45000, "Tạm hết");

        return dt;
    }

    private void BindDummyData()
    {
        gvKetQua.DataSource = TaoBangMau();
        gvKetQua.DataBind();
    }

    protected void btnTimKiem_Click(object sender, EventArgs e)
    {
        DataTable dt = TaoBangMau();
        string keyword = txtTuKhoa.Text.Trim().ToLower();
        string loai = ddlLoai.SelectedValue;

        DataRow[] rows = dt.Select(
            string.Format("(TenMon LIKE '%{0}%' OR MaMon LIKE '%{0}%') {1}",
            keyword.Replace("'", "''"),
            string.IsNullOrEmpty(loai) ? "" : $" AND Loai = '{loai.Replace("'", "''")}'"));

        DataTable dtResult = dt.Clone(); // clone cấu trúc
        foreach (DataRow r in rows)
            dtResult.ImportRow(r);

        // TODO: xử lý sắp xếp theo ddlSort nếu cần

        gvKetQua.DataSource = dtResult;
        gvKetQua.DataBind();

        lblThongBao.Text = "Tìm thấy " + dtResult.Rows.Count + " kết quả.";
    }
}

    </div>
</asp:Content>
