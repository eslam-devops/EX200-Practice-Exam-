### Question 1

---

#### 1) Original question (verbatim, English)

Using Red Hat Enterprise Linux 9, how would you create and enable a new 100MB swap partition on the /dev/sdb drive so that it automatically becomes active at every system boot?

Identify the partition: Use fdisk /dev/sdb to create a new partition of size 100MB.

Format the partition: Execute mkswap /dev/sdbX (replace X with the partition number).

Enable the swap: Use swapon /dev/sdbX to activate the swap partition immediately.

Automate at boot: Add an entry to /etc/fstab like /dev/sdbX swap swap defaults 0 0 to ensure it's activated at boot.

Verify with swapon --show and free -m.

---

#### 2) الترجمة بالعربي:

باستخدام نظام Red Hat Enterprise Linux 9، كيف يمكنك إنشاء وتمكين قسم swap جديد بحجم 100 ميغابايت على القرص /dev/sdb بحيث يتم تفعيله تلقائياً عند كل إقلاع للنظام؟

تحديد القسم: استخدم الأمر `fdisk /dev/sdb` لإنشاء قسم جديد بحجم 100 ميغابايت.

تهيئة القسم: نفّذ الأمر `mkswap /dev/sdbX` (استبدل X برقم القسم).

تفعيل الـ swap: استخدم الأمر `swapon /dev/sdbX` لتفعيل قسم الـ swap فوراً.

جعل العملية تلقائية عند الإقلاع: أضف سطراً إلى ملف `/etc/fstab` مثل `/dev/sdbX swap swap defaults 0 0` لضمان تفعيله عند الإقلاع.

تحقق باستخدام `swapon --show` و `free -m`.

---

#### 3) الحل والخطوات المفصلة (Solution — step-by-step)

📌 **ملخص قصير:** المطلوب نعمل partition جديد على `/dev/sdb` حجمه 100MB ونخليه swap، نفعّله فورًا ونضيفه في `/etc/fstab` عشان يشتغل تلقائي مع كل boot.

---

🔧 **الخطوات العملية:**

**1. نحدد القرص وندخل fdisk**

```bash
sudo fdisk /dev/sdb
```

💬 الخطوة دي بتفتح أداة `fdisk` عشان نقدر نعمل partition جديد.

داخل fdisk:

* اكتب `n` عشان تعمل partition جديد.
* اختار primary.
* اقبل الرقم الافتراضي لو هو أول partition على القرص.
* لما يسألك عن الـ size، اكتب `+100M`
* بعد ما تخلص، اكتب `t` لتغيير type واختار نوع **82** (Linux swap).
* في الآخر اكتب `w` لحفظ التغييرات.

---

**2. تهيئة القسم كـ swap**
⚠️ **تحذير:** الخطوة دي هتمسح أي داتا كانت موجودة على الـ partition ده.

```bash
sudo mkswap /dev/sdb1
```

💬 الأمر ده بيحوّل الـ partition الجديد لنظام swap جاهز للاستخدام.

---

**3. تفعيل الـ swap فوراً بدون reboot**

```bash
sudo swapon /dev/sdb1
```

💬 بيخلّي الـ swap يشتغل مباشرة على السيستم الحالي.

---

**4. نخليه يشتغل تلقائي مع كل boot**
افتح ملف `/etc/fstab` وأضف السطر التالي في آخر الملف:

```text
/dev/sdb1 swap swap defaults 0 0
```

💬 السطر ده بيقول للنظام إنه يفعّل الـ swap partition ده عند الإقلاع.

---

**5. تحقق من نجاح العملية:**

```bash
sudo swapon --show
```

مفروض تلاقي `/dev/sdb1` ظاهر في القائمة.

وكمان:

```bash
free -m
```

هتلاقي مساحة الـ swap زادت 100MB.

---

#### 4) الأخطاء الشائعة وازاي نحلها

* ❌ **نسيان تغيير الـ partition type لـ 82 (Linux swap)**
  🔧 حل:

  ```bash
  sudo fdisk /dev/sdb
  # استخدم t لتغيير النوع، ثم w للحفظ
  ```
* ❌ **نسيان إضافة السطر في /etc/fstab → الـ swap مش بيشتغل بعد reboot**
  🔧 تحقق من:

  ```bash
  cat /etc/fstab
  ```
* ❌ **أمر mkswap على partition مش متقسم أصلاً** → هيديك error.
  🔧 استخدم:

  ```bash
  lsblk
  ```

  للتأكد إن الـ partition موجود.
* ❌ **ملف /etc/fstab فيه خطأ syntax → الجهاز مش هيعمل boot صح**
  🔧 استخدم:

  ```bash
  sudo mount -a
  ```

  قبل الـ reboot لاختبار صحة الملف.
* ❌ **تفعيل swap على قرص مش متوصل أو مش موجود**
  🔧 شوف journal:

  ```bash
  journalctl -xe | grep swap
  ```

---

#### 5) الحاجات المرتبطة (Related topics / exam mapping)

* أوامر: `fdisk`, `partprobe`, `lsblk`, `mkswap`, `swapon`, `swapoff`
* ملفات مهمة: `/etc/fstab`
* RHCSA Objective: **"Manage local storage"** + **"Create and configure file systems"**

---

#### 6) سوال شبهه في بيئة عمل (Workplace scenario — similar question)

Scenario: You are the sysadmin at Company X. Your monitoring system shows that the application server sometimes runs out of RAM during peak hours. You are asked to add a 512MB swap partition on `/dev/vdb`, activate it immediately, and ensure it's persistent across reboots — but with **minimal downtime**.

✅ **Acceptance criteria:**

* Swap must be visible with `swapon --show`.
* Swap must survive reboot (still active).
* No data loss on existing partitions.

🔒 **Constraints:**

* No reboot allowed during configuration.
* Must not affect existing data on `/dev/vdb`.

💡 **Hints:**

* Use `fdisk` or `parted` to create the partition safely.
* Always test `/etc/fstab` entry using `mount -a`.

---

#### 7) تعديلات مقترحة لرفع/تخفيض مستوى الصعوبة (Suggested variations)

* 🟢 **أسهل:**

  * اعمل swap file بدل partition (باستخدام `fallocate` + `mkswap`).
  * فعّل swap بشكل يدوي فقط بدون إضافة في fstab.

* 🔴 **أصعب:**

  * اعمل LVM logical volume كـ swap بدل partition عادي.
  * غيّر الـ swapiness value في `/proc/sys/vm/swappiness` وخليه persistent.

---

#### 8) مراجعة سريعة وخطة تمرين (Quick review + practice plan)

📝 **تمارين سريعة:**

1. اعمل partition تاني بحجم 50MB وخليه swap مؤقت.

```bash
sudo mkswap /dev/sdb2
sudo swapon /dev/sdb2
```

2. جرّب تعطيل swap:

```bash
sudo swapoff /dev/sdb2
```

3. اختبر تعديل /etc/fstab بإضافة و حذف السطر ثم استخدم:

```bash
sudo mount -a
```

📚 **اقرأ:**

* `man fdisk`
* `man mkswap`

🔎 **ابحث عن:**
"RHEL9 create swap file vs swap partition"

---

#### 9) أي تحسينات أو تعليقات إضافية منك (Optional improvements I add)
---
### Question 2

---

#### 1) Original question (verbatim, English)

In Red Hat Enterprise Linux 9 (RHEL 9), how can you configure SELinux (Security-Enhanced Linux) to operate in 'permissive' mode? Provide a concise explanation of your steps.

To set SELinux to "permissive" mode in RHEL 9, edit the SELinux configuration file located at /etc/selinux/config. Change the line SELINUX=enforcing to SELINUX=permissive. This tells SELinux to log potential violations instead of enforcing policy, allowing all operations but reporting actions that would have been blocked in enforcing mode. After making this change, reboot the system or run setenforce 0 to apply the changes immediately. This mode is useful for troubleshooting and understanding SELinux policies without impacting system functionality.

---

#### 2) الترجمة بالعربي:

في نظام Red Hat Enterprise Linux 9، كيف يمكنك ضبط SELinux (لينكس المحسّن بالأمان) ليعمل في وضع "permissive"؟ قدّم شرحاً موجزاً لخطواتك.

لضبط SELinux على وضع "permissive" في RHEL 9، قم بتحرير ملف إعدادات SELinux الموجود في `/etc/selinux/config`، وقم بتغيير السطر `SELINUX=enforcing` إلى `SELINUX=permissive`. هذا الإعداد يجعل SELinux يسجّل الانتهاكات المحتملة بدلاً من تطبيق السياسة بشكل صارم، بحيث يسمح بكل العمليات لكنه يبلغ عن الأفعال التي كان سيمنعها في وضع التنفيذ. بعد إجراء التغيير، أعد تشغيل النظام أو نفّذ الأمر `setenforce 0` لتطبيق التغييرات فوراً. هذا الوضع مفيد لاستكشاف الأخطاء وفهم سياسات SELinux بدون التأثير على وظائف النظام.

---

#### 3) الحل والخطوات المفصلة (Solution — step-by-step)

📌 **ملخص قصير:** المطلوب نخلي SELinux يشتغل في وضع "permissive" عشان يسجّل الانتهاكات بس ومايمنعش العمليات. ده مفيد وقت الـ troubleshooting.

---

🔧 **الخطوات العملية:**

**1. تحقق من الوضع الحالي لـ SELinux**

```bash
getenforce
```

💬 ده بيطبع الوضع الحالي: Enforcing أو Permissive أو Disabled.

---

**2. تغييره فوراً بدون reboot (مؤقت)**

```bash
sudo setenforce 0
```

💬 الأمر ده بيحوّل SELinux لـ permissive مؤقتاً لحد الـ reboot الجاي.

تحقق بعد التغيير:

```bash
getenforce
# Expected output: Permissive
```

---

**3. تغييره بشكل دائم (Persistent)**
افتح ملف `/etc/selinux/config`:

```bash
sudo vi /etc/selinux/config
```

غيّر السطر:

```text
SELINUX=enforcing
```

إلى:

```text
SELINUX=permissive
```

💬 السطر ده بيتحدد وضع SELinux عند كل boot. التغيير هنا بيخلي الوضع Permissive بعد كل reboot.

---

**4. إعادة تشغيل النظام أو تطبيق التغيير فوراً**
لو عايز تطبقه دلوقتي بدون restart:

```bash
sudo setenforce 0
```

ولو هتعمل restart:

```bash
sudo reboot
```

---

#### 4) الأخطاء الشائعة وازاي نحلها

* ❌ **نسيان sudo أو تعديل الملف من غير صلاحيات root**
  🔧 استخدم:

  ```bash
  sudo -i
  ```

  أو:

  ```bash
  sudoedit /etc/selinux/config
  ```
* ❌ **نسيان إعادة تشغيل الجهاز أو استخدام setenforce → الوضع مايتغيرش بعد reboot**
  🔧 حل: تأكد إن الملف اتعدل:

  ```bash
  grep SELINUX= /etc/selinux/config
  ```
* ❌ **كتابة السطر غلط (مثلاً SELINUX=permissiv)** → السيستم هيرجع للوضع الافتراضي Enforcing
  🔧 راجع spelling كويس.
* ❌ **الاعتقاد إن setenforce بيغيره بشكل دائم** → لأ، ده بيأثر لحد الـ reboot بس.
* ❌ **عدم التحقق من حالة SELinux بعد التغيير**
  🔧 استخدم:

  ```bash
  sestatus
  ```

---

#### 5) الحاجات المرتبطة (Related topics / exam mapping)

* أوامر: `getenforce`, `setenforce`, `sestatus`
* ملف مهم: `/etc/selinux/config`
* RHCSA Objective: **"Configure and use SELinux"**

---

#### 6) سوال شبهه في بيئة عمل (Workplace scenario — similar question)

Scenario: You are troubleshooting why Apache is failing to serve files from `/srv/www`. Your manager asks you to temporarily put SELinux into permissive mode to confirm whether SELinux is blocking access.

✅ **Acceptance criteria:**

* SELinux must be in permissive mode (confirmed with `getenforce`).
* After test, SELinux should be put back to enforcing mode.

🔒 **Constraints:**

* No reboot allowed during troubleshooting.
* Must log SELinux events during the test for later review.

💡 **Hints:**

* Use `setenforce 0` to go permissive, then `setenforce 1` to revert.
* Use `journalctl -t setroubleshoot` or `ausearch -m avc` to review logs.

---

#### 7) تعديلات مقترحة لرفع/تخفيض مستوى الصعوبة (Suggested variations)

* 🟢 **أسهل:**

  * بس استخدم `getenforce` لعرض الوضع الحالي بدون تغييره.
  * جرّب `setenforce 0` فقط بدون تعديل الملف (مؤقت).

* 🔴 **أصعب:**

  * خلي السؤال يطلب تفعيل وضع permissive لعملية معينة باستخدام boolean مناسب (مثلاً httpd\_can\_network\_connect).
  * اطلب استخدام `semanage` لإدارة السياسات بدل تعديل ملف config.

---

#### 8) مراجعة سريعة وخطة تمرين (Quick review + practice plan)

📝 **تمارين سريعة:**

1. اعرض الوضع الحالي لـ SELinux.

```bash
getenforce
```

2. حوّل الوضع لـ permissive مؤقتاً ورجعه تاني لـ enforcing.

```bash
sudo setenforce 0
sudo setenforce 1
```

3. عدل `/etc/selinux/config` وخلي الوضع Permissive، اعمل reboot وتأكد إنه محفوظ.

📚 **اقرأ:**

* `man selinux`
* `man setenforce`

🔎 **ابحث عن:**
"Troubleshoot SELinux AVC denials RHEL 9"

---

#### 9) أي تحسينات أو تعليقات إضافية منك (Optional improvements I add)

💡 كان ممكن السؤال يطلب كمان ترجيع الوضع لـ enforcing بعد ما تخلص troubleshooting — لأن ده practice كويسة في البيئات الحقيقية عشان متسيبش السيستم في وضع أقل أمان.

---

تحب أجهزلك سؤال عملي أصعب عن SELinux فيه تعديل boolean أو إعادة label لمجلد (زي `restorecon -Rv` على مجلد الويب)؟ ده بيجي كتير في الامتحان.


💡 كان ممكن السؤال يطلب استخدام **UUID** بدل `/dev/sdbX` في `/etc/fstab` لأنه أفضل practice لتفادي مشاكل لو ترتيب الأقراص اتغير بعد reboot.

---

تحب أجهزلك مثال مشابه لكن باستخدام **swap file** بدل partition (لأن ده برضه جزء من امتحان RHCSA وبيجي كتير)؟
