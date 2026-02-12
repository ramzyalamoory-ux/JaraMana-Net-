import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

export default function JaraManaNetSystem() {
  const [user, setUser] = useState(null);
  const [customers, setCustomers] = useState([]);
  const [invoices, setInvoices] = useState([]);

  const [loginUser, setLoginUser] = useState("");
  const [loginPass, setLoginPass] = useState("");

  const [newCustomer, setNewCustomer] = useState({ name: "", phone: "", username: "", password: "" });
  const [newInvoice, setNewInvoice] = useState({ customer: "", amount: "", date: "", details: "", paid: false });

  const [editProfile, setEditProfile] = useState({ name: "", password: "" });
  const [lookupName, setLookupName] = useState("");
  const [lookupResult, setLookupResult] = useState([]);

  useEffect(() => {
    setCustomers(JSON.parse(localStorage.getItem("customers")) || []);
    setInvoices(JSON.parse(localStorage.getItem("invoices")) || []);
  }, []);

  useEffect(() => {
    localStorage.setItem("customers", JSON.stringify(customers));
  }, [customers]);

  useEffect(() => {
    localStorage.setItem("invoices", JSON.stringify(invoices));
  }, [invoices]);

  const handleLogin = () => {
    if (loginUser === "admin" && loginPass === "1234") {
      setUser({ role: "admin", name: "Eng Ramzy Amoory", username: "admin", password: "1234" });
      setEditProfile({ name: "Eng Ramzy Amoory", password: "1234" });
      return;
    }

    const found = customers.find(
      (c) => c.username === loginUser && c.password === loginPass
    );

    if (found) {
      setUser({ role: "customer", name: found.name, username: found.username, password: found.password });
      setEditProfile({ name: found.name, password: found.password });
    } else {
      alert("بيانات غير صحيحة");
    }
  };

  const addCustomer = () => {
    if (!newCustomer.name) return;
    setCustomers([...customers, newCustomer]);
    setNewCustomer({ name: "", phone: "", username: "", password: "" });
  };

  const addInvoice = () => {
    if (!newInvoice.customer || !newInvoice.amount) return;
    setInvoices([...invoices, newInvoice]);
    setNewInvoice({ customer: "", amount: "", date: "", details: "", paid: false });
  };

  const togglePaid = (index) => {
    const updated = [...invoices];
    updated[index].paid = !updated[index].paid;
    setInvoices(updated);
  };

  const saveProfile = () => {
    if (user.role === "admin") {
      setUser({ ...user, name: editProfile.name, password: editProfile.password });
    } else {
      const updatedCustomers = customers.map(c => c.username === user.username ? { ...c, name: editProfile.name, password: editProfile.password } : c);
      setCustomers(updatedCustomers);
      setUser({ ...user, name: editProfile.name, password: editProfile.password });
    }
    alert("تم تحديث البيانات بنجاح");
  };

  const checkInvoices = () => {
    const result = invoices.filter(i => i.customer.toLowerCase() === lookupName.toLowerCase());
    setLookupResult(result);
  };

  const totalPaid = invoices.filter((i) => i.paid).reduce((sum, i) => sum + Number(i.amount), 0);
  const totalUnpaid = invoices.filter((i) => !i.paid).reduce((sum, i) => sum + Number(i.amount), 0);

  // واجهة المدير
  if (user && user.role === "admin") {
    return (
      <div className="p-6">
        <h1 className="text-2xl font-bold mb-4">لوحة المدير - {user.name}</h1>

        <Card className="mb-6">
          <CardContent className="p-4 space-y-2">
            <h2 className="font-semibold">تعديل بياناتي</h2>
            <Input placeholder="الاسم" value={editProfile.name} onChange={(e) => setEditProfile({ ...editProfile, name: e.target.value })} />
            <Input type="password" placeholder="كلمة المرور" value={editProfile.password} onChange={(e) => setEditProfile({ ...editProfile, password: e.target.value })} />
            <Button onClick={saveProfile}>حفظ التغييرات</Button>
          </CardContent>
        </Card>

        <Card className="mb-6">
          <CardContent className="p-4 space-y-2">
            <h2 className="font-semibold">إضافة زبون</h2>
            <Input placeholder="اسم" value={newCustomer.name} onChange={(e) => setNewCustomer({ ...newCustomer, name: e.target.value })} />
            <Input placeholder="موبايل" value={newCustomer.phone} onChange={(e) => setNewCustomer({ ...newCustomer, phone: e.target.value })} />
            <Input placeholder="اسم دخول" value={newCustomer.username} onChange={(e) => setNewCustomer({ ...newCustomer, username: e.target.value })} />
            <Input placeholder="كلمة مرور" value={newCustomer.password} onChange={(e) => setNewCustomer({ ...newCustomer, password: e.target.value })} />
            <Button onClick={addCustomer}>إضافة</Button>
          </CardContent>
        </Card>

        <Card className="mb-6">
          <CardContent className="p-4 space-y-2">
            <h2 className="font-semibold">إضافة فاتورة</h2>
            <Input placeholder="اسم الزبون" value={newInvoice.customer} onChange={(e) => setNewInvoice({ ...newInvoice, customer: e.target.value })} />
            <Input placeholder="المبلغ" value={newInvoice.amount} onChange={(e) => setNewInvoice({ ...newInvoice, amount: e.target.value })} />
            <Input type="date" value={newInvoice.date} onChange={(e) => setNewInvoice({ ...newInvoice, date: e.target.value })} />
            <Input placeholder="تفاصيل" value={newInvoice.details} onChange={(e) => setNewInvoice({ ...newInvoice, details: e.target.value })} />
            <Button onClick={addInvoice}>إضافة فاتورة</Button>
          </CardContent>
        </Card>

        <Card className="mb-6">
          <CardContent className="p-4">
            <h2 className="font-semibold mb-2">تقارير الأرباح</h2>
            <p>إجمالي المدفوع: {totalPaid}</p>
            <p>إجمالي غير المدفوع: {totalUnpaid}</p>
          </CardContent>
        </Card>

        <h2 className="font-bold mb-2">كل الفواتير</h2>
        {invoices.map((inv, index) => (
          <Card key={index} className="mb-3">
            <CardContent className="p-3">
              <p><strong>{inv.customer}</strong></p>
              <p>المبلغ: {inv.amount}</p>
              <p>التاريخ: {inv.date}</p>
              <p>الحالة: {inv.paid ? "مدفوع" : "غير مدفوع"}</p>
              <Button onClick={() => togglePaid(index)}>تغيير الحالة</Button>
            </CardContent>
          </Card>
        ))}

        <Button onClick={() => setUser(null)}>تسجيل خروج</Button>
        <p className="text-xs mt-4">تم الإنشاء بواسطة Eng Ramzy Amoory</p>
        <p className="text-xs">الدعم الفني: 0959128944 / 0992417870</p>
      </div>
    );
  }

  // واجهة الزبون البسيطة بدون دخول
  return (
    <div className="min-h-screen flex flex-col items-center justify-center bg-gray-100">
      <Card className="w-96 p-6 mb-4">
        <CardContent className="space-y-4">
          <h2 className="text-xl font-bold text-center">JaraMana Net - الزبون</h2>
          <Input placeholder="أدخل اسمك" value={lookupName} onChange={(e) => setLookupName(e.target.value)} />
          <Button onClick={checkInvoices}>تحقق</Button>

          {lookupResult.length > 0 ? (
            <div className="mt-2">
              <p>فواتيرك:</p>
              {lookupResult.map((inv, idx) => (
                <Card key={idx} className="mb-2">
                  <CardContent>
                    <p>المبلغ: {inv.amount}</p>
                    <p>التاريخ: {inv.date}</p>
                    <p>الحالة: {inv.paid ? "مدفوع" : "غير مدفوع"}</p>
                  </CardContent>
                </Card>
              ))}
            </div>
          ) : (
            lookupName && <p className="mt-2">لا توجد فواتير على هذا الاسم</p>
          )}

          <div className="mt-2">
            <p>الدعم الفني:</p>
            <p>0959128944</p>
            <p>0992417870</p>
          </div>
        </CardContent>
      </Card>
      <p className="text-xs mt-4">تم الإنشاء بواسطة Eng Ramzy Amoory</p>
    </div>
  );
}
